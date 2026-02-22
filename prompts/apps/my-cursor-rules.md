# My Cursor Rules

- Use a professional and formal tone.
- Add spaces between Chinese and English text, in accordance with the well-known Chinese Copywriting Guidelines.
- Do not add numbering to headings; follow Markdown conventions.
- Avoid overly casual analogies, and do not attempt to explain concepts through metaphors.
- Maintain a high level of professionalism and competitiveness in the content; avoid redundancy and excessive repetition of obvious background, facts, concepts, or summaries.
- Write modern code using up-to-date APIs and actively maintained libraries, avoiding deprecated or poorly maintained tools.
- Write from a professional perspective; commonly used technical terms can be left in English without explanatory comments.
- Always use English for code, comments and docs but response in user's language.
- Always review `git diff` and inspect the latest 10 commit messages before committing.
- Don not derive state in `useEffect`; compute it during render (or `useMemo`).
- Don not sync state in `useEffect`; use a single source of truth (e.g., reducer).
- Don not turn user actions into effects; handle them in event handlers.
- You are in **YOLO mode**: commands may run without explicit user consent. For any action that could affect anything **outside the current workspace** (e.g., `brew install`, emptying Trash, changing global `git`/`ssh` configs, `fnm install`, `pnpm -g install`, etc.), **do not use any command-execution tools**. Provide guidance in text only; if you include commands, output them as **plain text** for the user to run manually.
- Always use the latest stable SDKs and APIs, and target the latest stable OS/platform versions. Do not assume what “latest” is—verify via web search before answering. Apply this consistently to Apple (iOS/macOS/watchOS/tvOS/visionOS), Android, React, and any related toolchains.
- Always add `#Preview { ... }` for SwiftUI views.
- ASCII diagrams must use ASCII-only characters. Chinese characters and emoji may break alignment in monospaced fonts.
