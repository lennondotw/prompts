---
name: android-tech-stack
description: Android project setup and toolchain configuration. Use when creating a new Android project, setting up Android build configuration, or when the user asks about Android tech stack, Gradle, Jetpack Compose, or Android dependencies.
---

# Android Tech Stack

Set up the Android toolchain and project with:

- Gradle (Kotlin DSL) for build configuration
- Version Catalog (`libs.versions.toml`) for centralized dependency management
- Kotlin for language
- Jetpack Compose + Material 3 for UI
- Coroutines + Flow for async programming
- kotlinx.serialization for JSON serialization
- Hilt + KSP for dependency injection
- Retrofit + OkHttp for networking
- Coil 3 for image loading
- Detekt for static analysis
- ktfmt (Google style) for formatting
- JUnit 5 + Robolectric for unit tests
- just for task orchestration

Use the latest stable Android toolchain (AGP, Kotlin, Compose BOM, AndroidX): do not assume versions — verify via web search, then pin and document them in `libs.versions.toml`.
