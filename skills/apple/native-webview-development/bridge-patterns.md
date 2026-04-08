# Swift ↔ JavaScript Bridge Patterns

## TypeScript Type Definitions

Add this to your project to type the `window.webkit` global:

```typescript
declare global {
  interface Window {
    webkit?: {
      messageHandlers: {
        [name: string]: {
          postMessage: (body: unknown) => void;
        };
      };
    };
    __native__?: {
      config: NativeConfig;
    };
  }
}

export const isNative = !!window.webkit?.messageHandlers?.bridge;

export function postMessage(msg: NativeMessage): void {
  if (isNative) {
    window.webkit!.messageHandlers.bridge.postMessage(msg);
  } else {
    console.log('[bridge:mock]', msg.type, msg);
  }
}
```

## Communication Channels

| Direction                      | API                               | Returns           | Min Version |
| ------------------------------ | --------------------------------- | ----------------- | ----------- |
| JS→Swift (fire-and-forget)     | `WKScriptMessageHandler`          | void              | iOS 8       |
| JS→Swift→JS (request/response) | `WKScriptMessageHandlerWithReply` | Promise           | iOS 14      |
| Swift→JS (legacy)              | `evaluateJavaScript`              | callback          | iOS 8       |
| Swift→JS (modern)              | `callAsyncJavaScript`             | async, typed args | iOS 14      |

## WKScriptMessageHandlerWithReply (recommended)

```swift
// Swift
config.userContentController.addScriptMessageHandler(
    handler, contentWorld: .defaultClient, name: "call"
)

func userContentController(
    _ ucc: WKUserContentController,
    didReceive message: WKScriptMessage,
    replyHandler: @escaping (Any?, String?) -> Void
) {
    // replyHandler MUST be called exactly once on every code path
    // (value, nil) for success, (nil, errorString) for failure
    replyHandler(result, nil)
}
```

```javascript
// JS — returns native Promise
const result = await window.webkit.messageHandlers.call.postMessage({ action: 'getToken' });
```

## Type Serialization (JS → Swift)

| JavaScript | Swift                       |
| ---------- | --------------------------- |
| String     | String                      |
| Number     | NSNumber (Int/Double)       |
| Boolean    | NSNumber (use `.boolValue`) |
| Array      | [Any]                       |
| Object     | [String: Any]               |
| null       | NSNull (not Swift nil)      |
| undefined  | postMessage silently fails  |
| Date       | Date                        |
| TypedArray | NOT supported (use base64)  |

## WKUserScript Injection Timing

| Timing             | When                                   | Use For                        |
| ------------------ | -------------------------------------- | ------------------------------ |
| `.atDocumentStart` | Before any page JS                     | Bridge setup, config injection |
| `.atDocumentEnd`   | After HTML parsed (≈ DOMContentLoaded) | DOM manipulation               |

Bridge initialization MUST use `atDocumentStart`.

## Retain Cycle (the famous bug)

`WKUserContentController.add(_:name:)` strongly retains the handler, creating a cycle:

```
ViewController → WKWebView → WKWebViewConfiguration
    → WKUserContentController → handler (= ViewController) ← CYCLE
```

Fix with weak proxy:

```swift
final class WeakMessageHandler: NSObject, WKScriptMessageHandler {
    weak var target: WKScriptMessageHandler?
    init(_ target: WKScriptMessageHandler) { self.target = target }
    func userContentController(_ ucc: WKUserContentController, didReceive message: WKScriptMessage) {
        target?.userContentController(ucc, didReceive: message)
    }
}
config.userContentController.add(WeakMessageHandler(self), name: "bridge")
```

## Teardown Sequence

```swift
webView.stopLoading()
webView.navigationDelegate = nil
webView.uiDelegate = nil
webView.configuration.userContentController.removeScriptMessageHandler(forName: "bridge")
webView.configuration.userContentController.removeAllUserScripts()
// Then nil out references, close window
```

## WebContent Process Crash Recovery

```swift
func webViewWebContentProcessDidTerminate(_ webView: WKWebView) {
    webView.reload()  // webView.url is still valid
}
```

Add a watchdog in `viewDidAppear` for cases where the callback doesn't fire.

## WKContentWorld — JS Namespace Isolation

| World            | Access                | Use For                       |
| ---------------- | --------------------- | ----------------------------- |
| `.page`          | Shared with page JS   | DO NOT inject privileged code |
| `.defaultClient` | Isolated from page JS | Bridge scripts (recommended)  |
| `.world(name:)`  | Custom namespace      | Multi-plugin scenarios        |

Worlds share the DOM but have isolated JS variables. XSS in `.page` cannot access `.defaultClient` bridge APIs.

## ATS for localhost dev server

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsLocalNetworking</key>
    <true/>
</dict>
```

Only allows localhost and `.local` domains. Must be DEBUG-only.

## Common Pitfalls

| Symptom                          | Cause                                       | Fix                            |
| -------------------------------- | ------------------------------------------- | ------------------------------ |
| ViewController never deallocates | Handler retain cycle                        | Weak proxy + manual removal    |
| JS Promise hangs forever         | replyHandler not called                     | Ensure called on every path    |
| `callAsyncJavaScript` error 5    | void function returns undefined             | Add `; null` at end            |
| White screen, no callback        | Process terminated silently                 | Watchdog in viewDidAppear      |
| `alert()` freezes page           | WKUIDelegate not implemented                | Implement all 3 dialog methods |
| JS injection silently fails      | WKAppBoundDomains without limitsNavigations | Must use both together         |
