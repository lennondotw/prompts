---
name: apple-tech-stack
description: Apple platform (iOS/macOS) project setup and toolchain configuration. Use when creating a new iOS or macOS project, setting up Xcode toolchain, or when the user asks about Apple tech stack, SwiftUI, XcodeGen, or Swift project setup.
---

# Apple Tech Stack

Set up the iOS / macOS toolchain and project with:

- XcodeGen for project generation
- SwiftLint for linting
- SwiftFormat for formatting
- just for task orchestration
- xcbeautify for build logs
- SwiftPM for shared modules consumed outside an Xcode entrypoint (if any)

Use the latest stable Apple SDK and toolchain (Xcode, Swift, platform SDKs): do not assume versions — verify via web search, then pin and document them in the repo.
