# DynamicIslandAndroid — Architecture

## 1. Założenie

Dynamic Island ma być jednym stale istniejącym obiektem UI, którego stan jest wyprowadzany z aktywności systemowych i aplikacyjnych. Nie projektujemy osobnych popupów dla połączeń, muzyki, timera itd.

## 2. High-level flow

```text
SYSTEM / APPS
      |
      v
+-----------------------+
|      EVENT SOURCES    |
| Telecom               |
| Notifications         |
| MediaSession          |
| Battery               |
| Bluetooth             |
| Navigation adapters   |
| Live Updates          |
+-----------+-----------+
            |
            v
+-----------------------+
|      ACTIVITY HUB     |
| normalize / classify  |
+-----------+-----------+
            |
            v
+-----------------------+
|   ISLAND STATE ENGINE |
| priority / lifecycle  |
| concurrency / restore |
+-----------+-----------+
            |
            v
+-----------------------+
|    PRESENTATION MODEL |
| geometry / content    |
+-----------+-----------+
            |
            v
+-----------------------+
|    MOTION ENGINE      |
| morph / spring        |
+-----------+-----------+
            |
            v
+-----------------------+
|    OVERLAY RENDERER   |
+-----------------------+
```

## 3. Warstwy

### core-domain
Czyste modele bez zależności od Android UI:
- `IslandActivity`
- `ActivityType`
- `ActivityPriority`
- `ActivityLifecycle`
- `IslandMode`
- `IslandAction`

### activity-sources
Adaptery źródeł:
- telecom
- notifications
- media
- battery
- bluetooth
- navigation
- live updates

Każdy adapter mapuje dane platformowe na wspólny `IslandActivity`.

### state-engine
Odpowiada za:
- selekcję aktywności dominującej
- aktywność pomocniczą / split
- restore poprzedniej aktywności
- transition intent
- suppress / hide

Nie powinien znać View/Compose.

### presentation
Buduje `IslandPresentationState`:
- mode
- width/height target
- corner radius target
- anchor positions
- primary content
- secondary content
- enabled actions

### motion
Interpoluje pomiędzy presentation states.

Animowane właściwości:
- size
- position
- radius
- content alpha
- content scale
- content translation
- anchors

### overlay
Jedyna warstwa odpowiedzialna za WindowManager / overlay lifecycle.

## 4. Telefonowanie

Telefonowanie jest źródłem aktywności P0.

```text
Android Telecom
      |
      v
InCallService
      |
      v
CallRepository
      |
      v
CallActivityMapper
      |
      v
IslandActivity(type = CALL)
      |
      v
State Engine
```

### CallRepository
Powinien być single source of truth dla aktywnych połączeń.

Nie przechowywać krytycznego call state wyłącznie w Composable lub overlay ViewModelu.

### Przykładowe mapowanie

- RINGING -> IncomingCallActivity
- DIALING -> DialingCallActivity
- CONNECTING -> ConnectingCallActivity
- ACTIVE -> ActiveCallActivity
- HOLDING -> HeldCallActivity
- DISCONNECTED -> transient end state -> remove

### UI rule

Jeśli urządzenie jest odblokowane i overlay jest aktywny, połączenie ma być obsługiwane przez wyspę bez wymuszenia pełnoekranowego UI.

Jeśli lockscreen / system wymaga pełnoekranowego incoming UI, używamy dedykowanego call screen zgodnie z Android Telecom.

## 5. Priorytety

Proponowana baza:

```text
100 incoming_call
95  second_incoming_call
90  active_call
80  navigation
70  timer
60  media
50  ride_delivery
40  download_progress
30  charging_bluetooth
10  generic_notification
```

Priorytet jest wejściem do algorytmu, nie sztywnym zachowaniem UI. State Engine może zachować aktywność drugorzędną w split island.

## 6. Concurrency

State engine przechowuje aktywne activity records.

Przykład:

```text
media ACTIVE
call INCOMING

=> primary = call
=> secondary = media
=> mode = EXPANDED/COMPACT CALL

call ACTIVE
=> primary = call
=> secondary = media

call ENDED
=> primary = media
=> morph back without reset
```

## 7. Overlay hit testing

Overlay nie może przechwytywać całego ekranu.

Interaktywna strefa powinna odpowiadać aktualnej geometrii wyspy oraz tylko tym obszarom, które faktycznie obsługują gest.

Poza wyspą dotyk powinien trafiać do aplikacji pod spodem, z wyjątkiem świadomie zaprojektowanego expanded dismiss behavior.

## 8. Cutout geometry

`CutoutRepository` wylicza:
- physical bounding rect
- center X
- top inset
- visual padding
- minimum island width

Presentation layer nie zna konkretnego modelu telefonu.

## 9. Proces i usługi

Należy rozdzielić:
- trwały stan domenowy
- foreground/service lifecycle
- UI lifecycle

Overlay można odtworzyć po restarcie bez utraty informacji o aktywnej rozmowie lub media session.

## 10. Error / fallback strategy

Każde źródło ma działać niezależnie.

Przykłady:
- brak roli dialera -> wyłączone pełne call controls, reszta aplikacji działa
- brak notification access -> media i call mogą działać dalej
- brak overlay permission -> pokaż ustawienia/onboarding zamiast crasha
- brak cutout info -> użyj profilu lub kalibracji

## 11. UI technology

Ustawienia: Jetpack Compose + Material 3.

Overlay: zacząć od prototypu Compose, ale utrzymać architekturę renderera tak, aby można było przejść na bardziej kontrolowany custom View/Canvas renderer, jeśli profilowanie pokaże problemy z latency, allocation lub morphingiem.

## 12. Testowalność

State Engine powinien być testowalny bez urządzenia.

Najważniejsze testy:
- incoming call preempts media
- zakończenie rozmowy przywraca media
- second incoming nie usuwa active call
- timer + media daje poprawny split
- stale notification jest usuwane
- animation target może zostać przerwany nowym stanem
- process recreation odtwarza prawidłowy presentation state
