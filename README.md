# DynamicIslandAndroid

Dynamic Island for Android designed as a real activity surface, not a static notification popup.

The goal is to recreate the *behavioral model* of iOS Dynamic Island on Android: one persistent UI object around the camera cutout that morphs between calls, media, timers, navigation, charging, Bluetooth, downloads and other live activities.

## Core principles

- one persistent island, not separate popups
- smooth morphing between states
- interruptible spring-based motion
- multi-activity support
- automatic cutout/camera positioning
- minimal obstruction of the app underneath
- phone calls are a P0 feature
- **Material 3 is the mandatory design system for the entire product**

## Material 3

All application UI is built around one Material 3 design system.

This includes:

- onboarding
- settings
- calibration
- dialer
- full-screen call UI
- permissions and configuration
- the Dynamic Island itself

The island remains a custom surface because of its cutout geometry, compact states and morphing behavior, but it still uses the same Material 3 color roles, typography, shapes, spacing, iconography, accessibility rules and motion tokens.

The project does **not** use a separate Cupertino/iOS design system. It recreates the Dynamic Island interaction model while keeping the Android implementation visually consistent with Material 3.

## Phone calls

The project is designed from the beginning around optional full phone integration using Android Telecom, `ROLE_DIALER` and `InCallService`.

Target behavior:

- unlocked phone: incoming call appears in the island
- answering does not force the user into a full-screen call UI
- active call remains accessible in the island while using other apps
- expanded island exposes mute, speaker/Bluetooth, DTMF, hold and hang-up controls
- lockscreen can still use a dedicated full-screen incoming-call UI when appropriate
- multiple calls / call waiting are represented as first-class island states

## Planned integrations

- Android Telecom / calls
- NotificationListenerService
- MediaSession
- timers
- battery / charging
- Bluetooth / headphones
- navigation
- downloads / progress activities
- Android Live Updates / ProgressStyle where available

## Stack

- Kotlin
- Jetpack Compose
- Compose Material 3
- central Material 3 design system / theme
- Coroutines + Flow
- DataStore
- Android Telecom
- NotificationListenerService
- MediaSession APIs
- WindowManager / DisplayCutout APIs

## Documentation

- [PRD.md](PRD.md) — product requirements and UX behavior
- [ARCHITECTURE.md](ARCHITECTURE.md) — system architecture and state flow
- [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) — mandatory Material 3 design rules and tokens
- [TASK.md](TASK.md) — implementation roadmap

## Product goal

The island should feel like part of Android itself rather than an app drawing a rectangle over the screen: fluid like Dynamic Island, but designed and implemented as a native Material 3 Android experience.
