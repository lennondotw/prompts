---
name: browser-mcp-workflow
description: Use Chrome DevTools MCP and Playwright MCP together for browser automation and frontend verification. Use when testing web apps, verifying UI changes, debugging network requests, or running browser-based tests.
---

# Browser MCP Workflow

Chrome DevTools MCP and Playwright MCP share the same persistent browser instance. They complement each other and can be used together.

## Shared Browser Setup

Both MCPs connect to the same Chrome instance via CDP (Chrome DevTools Protocol):

```json
// ~/.cursor/mcp.json
{
  "Chrome DevTools": {
    "command": "pnpm",
    "args": ["dlx", "chrome-devtools-mcp@latest", "--browserUrl=http://127.0.0.1:9233"]
  },
  "Playwright": {
    "command": "pnpm",
    "args": ["dlx", "@playwright/mcp@latest", "--cdp-endpoint=http://127.0.0.1:9233"]
  }
}
```

The user manages the browser lifecycle. If MCP connection fails, ask the user to start Chrome with remote debugging enabled—do not attempt to launch it yourself.

## Before You Start

**Create your own tab** to avoid conflicts with other agents using the same instance:

```
# Playwright: create new tab
browser_tabs action="new" url="https://example.com"

# Chrome DevTools: create new tab
new_page url="https://example.com"
```

Always `list_pages` / `browser_tabs action="list"` first to see existing tabs.

## Complementary Capabilities

| Task                              | Use                                                             |
| --------------------------------- | --------------------------------------------------------------- |
| Navigate, click, type, screenshot | Either (similar APIs)                                           |
| Performance profiling             | Chrome DevTools (`performance_start_trace`, `lighthouse_audit`) |
| Memory analysis                   | Chrome DevTools (`take_memory_snapshot`)                        |
| Detailed network inspection       | Chrome DevTools (`get_network_request` with body/headers)       |
| Select dropdown options           | Playwright (`browser_select_option`)                            |
| Run automation scripts            | Playwright (`browser_run_code`)                                 |

## Common Verification Workflow

1. **Create tab**: `new_page` or `browser_tabs action="new"`
2. **Navigate**: `navigate_page` or `browser_navigate`
3. **Wait for content**: `wait_for` or `browser_wait_for`
4. **Check network**: `list_network_requests` - verify API calls and status codes
5. **Check console**: `list_console_messages` - catch JS errors
6. **Screenshot**: `take_screenshot` - visual confirmation
7. **Cleanup**: Close your tab when done

## Quick Reference

### Chrome DevTools MCP

- `navigate_page type="url" url="..."` / `type="reload"`
- `list_pages` / `select_page`
- `click uid="..."` (requires `take_snapshot` first for uid)
- `list_network_requests resourceTypes=["fetch","xhr"]`
- `evaluate_script function="() => { return ... }"`

### Playwright MCP

- `browser_navigate url="..."`
- `browser_tabs action="list|new|select|close"`
- `browser_click element="..." ref="..."` (from snapshot)
- `browser_network_requests`
- `browser_evaluate expression="..."`

## Tips

- Take a snapshot before clicking to get element refs/uids
- Use `includePreservedRequests: true` to see requests from previous navigations
- Filter network by `resourceTypes: ["fetch", "xhr"]` for API calls only
- Check console messages for JS errors after page load
