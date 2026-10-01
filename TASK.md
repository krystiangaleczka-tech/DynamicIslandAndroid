# DynamicIslandAndroid — TASKS

## Zasady

Każdy task powinien kończyć się testowalnym rezultatem. Najpierw budujemy fundament i telefonowanie, potem integracje dodatkowe.

**Material 3 jest obowiązkowym design systemem całego produktu.** Każdy task dotykający UI musi używać wspólnych tokenów, komponentów i zasad z `DESIGN_SYSTEM.md`. Nie wolno tworzyć lokalnego, niezależnego stylowania bez uzasadnionego wyjątku.

## Faza 0 — fundament repo i architektury

### TASK 1 — bootstrap aplikacji
- Kotlin + Gradle
- min/target SDK dobrane do aktualnego Androida
- Jetpack Compose
- Compose Material 3
- centralny `MaterialTheme`
- light/dark theme
- wspólny system color/typography/shape tokens
- Coroutines + Flow
- DataStore
- podstawowa struktura modułów

**Done when:** aplikacja buduje się i uruchamia, istnieje ekran ustawień, podstawowe moduły core oraz jeden wspólny Material 3 theme używany przez UI.

### TASK 1A — Material 3 Design System foundation
Zaimplementować techniczną warstwę design systemu zgodną z `DESIGN_SYSTEM.md`:
- semantic color roles
- typography tokens
- shape tokens
- spacing/sizing tokens
- icon rules
- reusable Material 3 components
- interaction/state layers
- accessibility defaults
- motion tokens
- preview/test screen komponentów

**Done when:** nowe ekrany i UI wyspy mogą korzystać z design tokens bez hardcodowania wartości wizualnych w feature modules.

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
- renderer przyjmuje theme/design tokens zamiast lokalnych wartości UI

### TASK 6 — IslandGeometry
Animowane parametry:
- width
- height
- cornerRadius
- x/y
- content anchors
- alpha
- scale
- geometry defaults oparte o wspólne sizing/shape tokens

### TASK 7 — Motion / Morph Engine
- spring transitions
- interruptible animations
- shared continuity
- animacja layoutu zamiast swapowania widoków
- bez 1-frame flash
- Material 3 motion jako baza
- własne spring/morph tokens tylko jako jawne rozszerzenie design systemu

### TASK 8 — gesty
- tap
- long press
- swipe
- collapse po tap poza expanded
- gesture arbitration
- spójny interaction feedback zgodny z Material 3

## Faza 2 — telefonowanie P0

### TASK 9 — ROLE_DIALER onboarding
- RoleManager
- oficjalny systemowy request
- stan roli w ustawieniach
- fallback, gdy użytkownik odmówi
- onboarding UI w Material 3

### TASK 10 — ACTION_DIAL / minimal dialer shell
- obsługa ACTION_DIAL
- prosty ekran numeru / dialera
- spełnienie wymagań roli domyślnego dialera
- pełny ekran dialera w Material 3
- standardowe Material 3 components tam, gdzie pasują

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
- UI wyspy korzysta ze wspólnych color/type/icon/state tokens Material 3

### TASK 13 — active call compact UI
- nazwa/numer
- timer
- zakończ
- tap -> expanded
- utrzymanie widoku podczas używania innych aplikacji
- zgodność z Material 3 semantics i accessibility

### TASK 14 — expanded call controls
- mute
- speaker
- audio endpoint / Bluetooth
- hang up
- keypad entry point
- hold/resume, jeśli dostępne
- Material 3 iconography, semantic colors i interaction states

### TASK 15 — DTMF keypad
- cyfry 0–9
- * / #
- poprawne start/stop DTMF
- overlay interaction
- Material 3 typography, sizing i touch targets

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
- full-screen Call UI spójny z Material 3

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
- Material 3 typography, icons, semantic states i design tokens

## Faza 4 — system activities

### TASK 23 — battery / charging
- podłączenie ładowarki
- poziom baterii
- transient animation
- auto collapse
- semantic colors z design systemu

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
- zachowanie wspólnego Material 3 token systemu w obu częściach

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

### TASK 32 — visual fidelity + Material 3 audit
- spacing zgodny z design tokens
- Material 3 typography
- Material 3 semantic colors
- Material 3 iconography
- shape/radius tokens
- state layers / pressed / focused / disabled
- motion tokens oraz island-specific spring extensions
- camera integration
- compact/expanded proportions
- dark island rendering jako kontrolowany wyjątek surface color
- accessibility contrast
- font scale / display size
- usunięcie przypadkowych hardcodowanych wartości UI

**Done when:** UI całej aplikacji i wyspy przechodzi checklistę z `DESIGN_SYSTEM.md` i nie istnieje drugi, niezależny system wizualny.

### TASK 33 — onboarding permissions
Kolejność i UX dla:
- overlay
- notification access
- default dialer
- Bluetooth / nearby devices, jeśli wymagane
- battery optimizations only if necessary
- wszystkie własne ekrany onboardingu w Material 3

### TASK 34 — settings UX
Pełny Material 3:
- General
- Position & calibration
- Calls
- Media
- Notifications
- Gestures
- Appearance
- Advanced
- Privacy

Wymagania:
- standardowe Material 3 components przed custom components
- wspólny `MaterialTheme`
- light/dark
- opcjonalny dynamic color
- responsive/adaptive layouts
- żadnych lokalnych kopii theme/tokenów

### TASK 35 — test matrix
Minimum:
- Samsung Galaxy S26 Ultra
- urządzenie Pixel z centralnym cutoutem
- różne density / font scale
- light/dark theme
- dynamic color on/off
- accessibility contrast/semantics
- lockscreen
- Bluetooth headset
- media + call
- second incoming call
- process kill / restart
- notification spam

## Priorytet implementacji

Najpierw: `1 -> 1A -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8`.

Następnie cały blok telefonu: `9 -> 18`.

Dopiero potem media i pozostałe integracje.

Powód: telefonowanie jest kluczowym wyróżnikiem produktu i musi wpływać na architekturę od początku, a Material 3 musi istnieć przed implementacją kolejnych ekranów, aby UI nie rozjechał się pomiędzy feature'ami.