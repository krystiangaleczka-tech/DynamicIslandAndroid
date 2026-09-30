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
- Material 3 for settings screens
- Coroutines + Flow
- DataStore
- Android Telecom
- NotificationListenerService
- MediaSession APIs
- WindowManager / DisplayCutout APIs

## Documentation

- [PRD.md](PRD.md) — product requirements and UX behavior
- [ARCHITECTURE.md](ARCHITECTURE.md) — system architecture and state flow
- [TASK.md](TASK.md) — implementation roadmap

## Product goal

The island should feel like part of the operating system rather than an app drawing a rectangle over the screen.
