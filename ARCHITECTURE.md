# DynamicIslandAndroid — Architecture

## 1. Założenie

Dynamic Island ma być jednym stale istniejącym obiektem UI, którego stan jest wyprowadzany z aktywności systemowych i aplikacyjnych. Nie projektujemy osobnych popupów dla połączeń, muzyki, timera itd.

Cały produkt korzysta z jednego systemu projektowego: **Material 3**. Dotyczy to zarówno zwykłych ekranów aplikacji, jak i niestandardowego renderera wyspy.

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
| MATERIAL 3 TOKENS     |
| color/type/shape      |
| spacing/motion/state  |
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

### design-system
Jedno źródło prawdy dla UI. Implementuje Material 3 i wystawia tokeny oraz reusable components.

Odpowiada za:
- `MaterialTheme`
- semantic color roles
- typography
- shapes
- spacing / sizing
- iconography
- state layers
- accessibility defaults
- motion tokens
- island-specific extensions, które nadal należą do wspólnego systemu

Feature modules nie mogą definiować własnych niezależnych theme/token sets.

### presentation
Buduje `IslandPresentationState`:
- mode
- width/height target
- corner radius target
- anchor positions
- primary content
- secondary content
- enabled actions
- semantic presentation roles pobierane z design systemu

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

Motion Engine korzysta z Material 3 motion jako bazy. Specjalne spring/morph parametry wyspy są centralnymi tokenami design systemu, nie wartościami wpisywanymi lokalnie w feature UI.

### overlay
Jedyna warstwa odpowiedzialna za WindowManager / overlay lifecycle.

Renderer overlayu przyjmuje gotowy presentation state oraz design tokens. Nie powinien samodzielnie definiować kolorów, fontów, radiusów lub spacingu.

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

Zarówno expanded call island, jak i pełny call screen korzystają z tego samego Material 3 design systemu.

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

Material 3 interaction feedback i accessibility semantics obowiązują również dla overlay controls.

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

Fallback UI również musi korzystać z Material 3.

## 11. UI technology

### Ekrany aplikacji

Jetpack Compose + Compose Material 3.

Wszystkie zwykłe ekrany korzystają z jednego `MaterialTheme`, wspólnych semantic colors, typography, shapes i reusable components z modułu design-system.

### Dynamic Island

Overlay zaczynamy od prototypu Compose, ale utrzymujemy architekturę renderera tak, aby można było przejść na bardziej kontrolowany custom View/Canvas renderer, jeśli profilowanie pokaże problemy z latency, allocation lub morphingiem.

Zmiana technologii renderera **nie może oznaczać zmiany design systemu**. Custom renderer musi otrzymywać te same Material 3 tokens co Compose UI.

Wyspa pozostaje czarną / bardzo ciemną kapsułą, gdy jest to wymagane do wizualnego połączenia z cutoutem. Ten surface color jest kontrolowanym wyjątkiem w ramach Material 3 theme, a nie osobnym stylem Cupertino.

## 12. Material 3 architecture rules

1. Material 3 jest jedynym bazowym językiem projektowym aplikacji.
2. Standardowy komponent Material 3 ma pierwszeństwo przed custom componentem, jeśli spełnia wymaganie UX.
3. Custom components są dozwolone dla Dynamic Island i specyficznych interakcji, ale używają tych samych tokenów.
4. Brak hardcodowanych feature-specific kolorów, fontów, radiusów i spacingu bez jawnego wpisania do design systemu.
5. Light/dark theme dotyczy ekranów aplikacji; wyspa może używać kontrolowanego dark surface niezależnie od motywu.
6. Dynamic color jest opcjonalnym źródłem theme colors dla aplikacji, nie może niszczyć czytelności wyspy.
7. Accessibility, semantic roles, contrast i font scaling są częścią design systemu, nie końcowym polish pass.
8. Szczegółowe reguły znajdują się w `DESIGN_SYSTEM.md`.

## 13. Testowalność

State Engine powinien być testowalny bez urządzenia.

Najważniejsze testy:
- incoming call preempts media
- zakończenie rozmowy przywraca media
- second incoming nie usuwa active call
- timer + media daje poprawny split
- stale notification jest usuwane
- animation target może zostać przerwany nowym stanem
- process recreation odtwarza prawidłowy presentation state
- UI nie omija centralnych Material 3 tokens
- light/dark/dynamic-color variants zachowują czytelność i semantics
