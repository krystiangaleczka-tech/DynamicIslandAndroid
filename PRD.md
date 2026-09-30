# DynamicIslandAndroid — PRD

## 1. Cel produktu

Zbudować dla Androida dynamiczną wyspę możliwie bliską modelowi działania iOS Dynamic Island — nie jako statyczny popup, ale jako stale dostępny, animowany system aktywności działający wokół wycięcia / otworu kamery.

Produkt ma priorytetowo poprawiać codzienne UX na Androidzie, szczególnie na urządzeniach z centralnym otworem kamery, np. Samsung Galaxy S26 Ultra.

## 2. Główna zasada produktu

Wyspa nie jest zbiorem osobnych popupów. Jest jednym trwałym obiektem UI, który płynnie przechodzi między stanami i reprezentuje aktualne aktywności systemowe.

Model:

`EVENT -> ACTIVITY -> STATE -> LAYOUT -> MOTION -> INTERACTION -> NEXT STATE`

## 3. Najważniejszy wyróżnik: telefonowanie jako funkcja P0

Obsługa rozmów telefonicznych ma być częścią rdzenia aplikacji.

### 3.1 Docelowe zachowanie

- Telefon odblokowany: połączenie przychodzące pojawia się w wyspie.
- Odbieranie nie powinno automatycznie przenosić użytkownika do pełnoekranowego widoku rozmowy.
- Po odebraniu rozmowa pozostaje w Dynamic Island podczas używania innych aplikacji.
- Tap lub long press rozwija kontrolki rozmowy.
- Pełny ekran rozmowy jest opcjonalny i otwierany dopiero na żądanie użytkownika.
- Telefon zablokowany: pełnoekranowy incoming-call UI jest dozwolony i pożądany.

### 3.2 Integracja systemowa

Tryb pełny ma korzystać z:

- `ROLE_DIALER`
- `InCallService`
- Android Telecom API
- `Call` / stanów połączenia
- audio endpoint / routing API dla speaker / earpiece / Bluetooth

Aplikacja w trybie pełnym musi spełniać wymagania domyślnej aplikacji telefonu, w tym obsługę `ACTION_DIAL` i kompletnego interfejsu in-call.

### 3.3 Stany rozmowy

Minimum:

- `incoming`
- `dialing`
- `connecting`
- `active`
- `held`
- `secondIncoming`
- `conference`
- `disconnected`

### 3.4 Kontrolki rozmowy

W expanded island:

- odbierz
- odrzuć
- zakończ
- mute / unmute
- speaker
- wybór endpointu audio / Bluetooth
- hold / resume
- DTMF keypad
- czas rozmowy
- nazwa / numer kontaktu
- przełączanie pomiędzy wieloma połączeniami

### 3.5 Połączenia alarmowe

Połączenia alarmowe pozostają obsługiwane zgodnie z ograniczeniami Androida i systemowym dialerem. Aplikacja nie może zakładać przejęcia emergency calling.

## 4. Dynamic Island — stany UI

Podstawowe stany wyspy:

1. `Idle`
2. `Compact`
3. `Expanded`
4. `Split`
5. `Transient`
6. `Hidden/Suppressed`

Wyspa ma zachowywać ciągłość wizualną pomiędzy stanami.

Nie wolno implementować przejść jako: ukryj poprzedni widok -> pokaż nowy widok.

Wymagane jest morphing UI: szerokość, wysokość, promień narożników, pozycje elementów, scale, alpha i content transform muszą animować się jako jedna spójna transformacja.

## 5. Multi-activity

System musi obsługiwać kilka równoległych aktywności.

Przykłady:

- rozmowa + timer
- rozmowa + muzyka
- nawigacja + muzyka
- timer + media
- połączenie oczekujące podczas aktywnej rozmowy

Przykładowa kolejność priorytetów:

1. incoming / active call
2. navigation
3. timer
4. media
5. download / transfer
6. ride / delivery live activity
7. charging / Bluetooth / system transient
8. zwykłe notification activity

Priorytet nie może usuwać aktywności niższego poziomu. Po zakończeniu aktywności dominującej wyspa ma płynnie wrócić do poprzedniego stanu.

## 6. Motion system

Animacje są funkcją pierwszej klasy.

Wymagania:

- spring-based transitions
- brak gwałtownych layout jumps
- animowany morph szerokości / wysokości
- animowany corner radius
- crossfade tylko jako element pomocniczy, nigdy jako główny mechanizm przejścia
- shared element continuity
- gesty powiązane z bieżącym stanem animacji
- możliwość przerwania animacji i płynnego przejścia do kolejnego stanu

## 7. Gesty

Minimum:

- tap: otwarcie / rozwinięcie aktywności
- long press: expanded controls
- swipe: schowanie, przełączenie aktywności lub powrót do compact — zależnie od kontekstu
- tap poza expanded island: collapse

Gesty nie mogą blokować interakcji z aplikacją pod overlayem poza faktycznym obszarem wyspy.

## 8. Integracje P0/P1/P2

### P0

- overlay + pozycjonowanie przy aparacie
- Island State Engine
- Motion / Morph Engine
- gesty
- telefonowanie przez `ROLE_DIALER` + `InCallService`
- Notification Listener
- MediaSession
- persistence / ustawienia

### P1

- muzyka / podcasty
- timer
- ładowanie / bateria
- Bluetooth / słuchawki
- nawigacja
- aktywne pobieranie / transfer
- Android Live Updates / ProgressStyle, gdy dostępne

### P2

- ride / delivery adapters
- rozbudowane multi-activity layouts
- per-app customization
- profile urządzeń / ręczna kalibracja cutoutu
- dodatkowe widgety systemowe

## 9. Media

Źródła:

- `MediaSessionManager`
- `MediaController`
- informacje z notification listenera, jeśli potrzebne

Compact:

- artwork / icon
- podstawowy status odtwarzania
- subtelna animacja waveform / equalizer

Expanded:

- tytuł
- wykonawca
- artwork
- play / pause
- previous / next, jeśli wspierane
- progress

## 10. Notifications / Live Activities

`NotificationListenerService` jest głównym źródłem kontekstu dla aktywności innych aplikacji.

Aplikacja ma mapować powiadomienia na typowane aktywności zamiast tylko kopiować treść powiadomienia.

Przykład:

`notification -> classifier/adapter -> RideActivity(eta=3 min) -> island state`

Na wspieranych wersjach Androida należy korzystać także z Live Updates / progress-style semantics, jeśli aplikacja źródłowa je wystawia.

## 11. Aparat / cutout

Pozycja wyspy nie może być hardcodowana wyłącznie pod jeden telefon.

Wymagane:

- `DisplayCutout`
- bounding rect cutoutu
- safe insets
- automatyczne obliczenie `cameraCenterX`
- profil urządzenia jako fallback
- ręczna kalibracja jako ostateczny fallback

Celem jest optyczne połączenie czarnego otworu aparatu z wyspą w jeden obiekt.

## 12. Overlay

Bazowy renderer może korzystać z `TYPE_APPLICATION_OVERLAY` / `SYSTEM_ALERT_WINDOW`.

Ograniczenie: zwykła aplikacja nie ma pełnej kontroli nad krytycznymi warstwami System UI. Produkt ma być projektowany tak, aby działał poprawnie mimo tego ograniczenia, bez założenia uprawnień OEM/root.

## 13. Accessibility

Nie używać `AccessibilityService` w MVP, jeśli dana funkcja jest możliwa publicznym API.

Accessibility można dodać tylko dla konkretnej funkcji, której nie da się wiarygodnie zrealizować inaczej, po osobnej analizie zgodności z Google Play.

## 14. Ustawienia aplikacji

Ekrany ustawień wykonujemy w Material 3.

Minimum:

- włącz / wyłącz Dynamic Island
- pozycja i kalibracja
- rozmiar compact / expanded
- gesty
- per-feature toggles
- konfiguracja telefonu / roli dialera
- media
- timer
- Bluetooth
- bateria / charging
- nawigacja
- prywatność i uprawnienia

Sama wyspa ma własny motion/design system i nie powinna wyglądać jak standardowy komponent Material 3.

## 15. Stack

Preferowany:

- Kotlin
- Jetpack Compose dla ustawień i ekranów aplikacji
- Compose lub wyspecjalizowany custom renderer dla overlayu, zależnie od wyników prototypu wydajnościowego
- Coroutines + Flow
- DataStore
- Android Telecom
- NotificationListenerService
- MediaSession APIs
- WindowManager / DisplayCutout APIs

## 16. Wymagania jakościowe

- 60/120 Hz friendly
- minimalny input latency
- zero widocznego migania przy zmianie aktywności
- brak 1-frame flash przed pojawieniem się treści
- odporność na recreation procesu / service restart
- stan rozmowy nie może zależeć tylko od UI
- żadna ważna aktywność nie może zniknąć przez utratę focusu aplikacji
- ograniczone zużycie baterii

## 17. Kryteria MVP

MVP jest ukończone, gdy:

1. wyspa jest prawidłowo ustawiona wokół cutoutu,
2. morphing pomiędzy Idle/Compact/Expanded wygląda płynnie,
3. tap/long-press/swipe działają stabilnie,
4. aktywne połączenie telefoniczne może być prowadzone z poziomu wyspy w trybie domyślnego dialera,
5. użytkownik po odebraniu rozmowy może korzystać z innych aplikacji bez obowiązkowego pełnoekranowego Call UI,
6. media są wykrywane i sterowane,
7. co najmniej dwa typy aktywności mogą współistnieć,
8. restart procesu nie pozostawia martwego overlayu ani niespójnego stanu.

## 18. Cel UX

Użytkownik ma mieć wrażenie, że Dynamic Island jest częścią systemu operacyjnego, a nie aplikacją wyświetlającą prostokąt nad ekranem.