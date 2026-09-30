# DynamicIslandAndroid — TASKS

## Zasady

Każdy task powinien kończyć się testowalnym rezultatem. Najpierw budujemy fundament i telefonowanie, potem integracje dodatkowe.

## Faza 0 — fundament repo i architektury

### TASK 1 — bootstrap aplikacji
- Kotlin + Gradle
- min/target SDK dobrane do aktualnego Androida
- Jetpack Compose
- Material 3 dla ekranów ustawień
- Coroutines + Flow
- DataStore
- podstawowa struktura modułów

**Done when:** aplikacja buduje się i uruchamia, istnieje ekran ustawień i podstawowe moduły core.

### TASK 2 — model domenowy IslandActivity
Zdefiniować wspólny model aktywności:
- id
- type
- priority
- lifecycle
- compact payload
- expanded payload
- actions
- timestamp / freshness

**Done when:** state engine może operować bez zależności od konkretnego źródła danych.

### TASK 3 — Island State Engine
Stany:
- Idle
- Compact
- Expanded
- Split
- Transient
- Suppressed

Obsłużyć:
- priorytety
- kolejkę aktywności
- restore poprzedniego stanu
- concurrency
- preemption

**Done when:** testy jednostkowe pokrywają przejścia i priorytety.

### TASK 4 — cutout / camera geometry
- DisplayCutout
- bounding rect
- safe insets
- cameraCenterX
- profile fallback
- ręczna kalibracja

**Done when:** wyspa poprawnie otacza centralny otwór kamery na docelowym urządzeniu i ma fallback dla innych urządzeń.

## Faza 1 — overlay i motion

### TASK 5 — overlay host
- TYPE_APPLICATION_OVERLAY
- SYSTEM_ALERT_WINDOW flow
- touch region tylko w obrębie wyspy
- lifecycle service
- attach/detach bez flicker

### TASK 6 — IslandGeometry
Animowane parametry:
- width
- height
- cornerRadius
- x/y
- content anchors
- alpha
- scale

### TASK 7 — Motion / Morph Engine
- spring transitions
- interruptible animations
- shared continuity
- animacja layoutu zamiast swapowania widoków
- bez 1-frame flash

### TASK 8 — gesty
- tap
- long press
- swipe
- collapse po tap poza expanded
- gesture arbitration

## Faza 2 — telefonowanie P0

### TASK 9 — ROLE_DIALER onboarding
- RoleManager
- oficjalny systemowy request
- stan roli w ustawieniach
- fallback, gdy użytkownik odmówi

### TASK 10 — ACTION_DIAL / minimal dialer shell
- obsługa ACTION_DIAL
- prosty ekran numeru / dialera
- spełnienie wymagań roli domyślnego dialera

### TASK 11 — InCallService core
- rejestracja InCallService
- CallRepository
- obserwowanie aktualnych Call objects
- mapowanie stanów Android Telecom do domeny aplikacji

### TASK 12 — incoming call in island
- caller identity
- accept
- reject
- kompaktowy incoming UI
- transition incoming -> active
- telefon odblokowany: bez wymuszonego full-screen Call UI

### TASK 13 — active call compact UI
- nazwa/numer
- timer
- zakończ
- tap -> expanded
- utrzymanie widoku podczas używania innych aplikacji

### TASK 14 — expanded call controls
- mute
- speaker
- audio endpoint / Bluetooth
- hang up
- keypad entry point
- hold/resume, jeśli dostępne

### TASK 15 — DTMF keypad
- cyfry 0–9
- * / #
- poprawne start/stop DTMF
- overlay interaction

### TASK 16 — multiple calls
- active + held
- second incoming
- swap
- call waiting
- conference representation
- split / stacked island layout

### TASK 17 — locked-screen behavior
- incoming full-screen UI tylko gdy faktycznie potrzebny
- poprawne zachowanie na lockscreen
- brak konfliktu z odblokowanym overlay mode

### TASK 18 — call resilience
- process recreation
- service reconnect
- call state restore
- rotacje / display changes
- Bluetooth disconnect/reconnect

## Faza 3 — notifications i media

### TASK 19 — NotificationListenerService
- permission flow
- parser
- source app metadata
- lifecycle
- debounce/dedup

### TASK 20 — activity adapters
Warstwa mapująca notifications na typowane aktywności.

Przykłady:
- ride
- delivery
- download
- navigation
- timer-like progress

### TASK 21 — MediaSession integration
- active sessions
- metadata
- artwork
- playback state
- play/pause
- next/previous
- seek, jeśli dostępny

### TASK 22 — media island UI
- compact media
- waveform/equalizer animation
- expanded player
- progress

## Faza 4 — system activities

### TASK 23 — battery / charging
- podłączenie ładowarki
- poziom baterii
- transient animation
- auto collapse

### TASK 24 — Bluetooth / headphones
- connected/disconnected
- urządzenie audio
- transient event
- współpraca z aktywną rozmową

### TASK 25 — timer
- wykrywanie / własny adapter
- countdown
- compact / expanded
- coexistence z media/call

### TASK 26 — navigation
- parser/adapters dla wspieranych źródeł
- next maneuver
- distance/ETA jeśli dostępne
- priorytet nad media

### TASK 27 — Live Updates / ProgressStyle
- obsługa semantics nowych Android notifications
- progress activities
- degradation na starszych wersjach

## Faza 5 — multi-activity i polish

### TASK 28 — split island
- dwie równoległe aktywności
- focus switching
- compact priority rules
- morph split <-> expanded

### TASK 29 — activity history / restore
- powrót do poprzedniej aktywności po zakończeniu dominującej
- bez wizualnego resetu

### TASK 30 — per-app rules
- enable/disable
- priority overrides
- notification classification overrides

### TASK 31 — performance pass
- frame timing
- 60/120 Hz
- allocations
- battery
- wakeups
- overlay invalidation

### TASK 32 — visual fidelity pass
- spacing
- typography
- easing/springs
- camera integration
- compact/expanded proportions
- dark-only island rendering

### TASK 33 — onboarding permissions
Kolejność i UX dla:
- overlay
- notification access
- default dialer
- Bluetooth / nearby devices, jeśli wymagane
- battery optimizations only if necessary

### TASK 34 — settings UX
Material 3:
- General
- Position & calibration
- Calls
- Media
- Notifications
- Gestures
- Appearance
- Advanced
- Privacy

### TASK 35 — test matrix
Minimum:
- Samsung Galaxy S26 Ultra
- urządzenie Pixel z centralnym cutoutem
- różne density / font scale
- lockscreen
- Bluetooth headset
- media + call
- second incoming call
- process kill / restart
- notification spam

## Priorytet implementacji

Najpierw: `1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8`

Następnie cały blok telefonu: `9 -> 18`.

Dopiero potem media i pozostałe integracje.

Powód: telefonowanie jest kluczowym wyróżnikiem produktu i musi wpływać na architekturę od początku, a nie być doklejone później.