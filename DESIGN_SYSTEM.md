# DynamicIslandAndroid — Material 3 Design System

## 1. Cel

Ten dokument definiuje obowiązkowy system projektowy całej aplikacji DynamicIslandAndroid.

Bazą jest **Material 3**. Nie tworzymy osobnego stylu dla ustawień, dialera, ekranu połączenia i Dynamic Island. Wszystkie powierzchnie należą do jednego systemu wizualnego i korzystają ze wspólnych tokenów.

Dynamic Island jest niestandardowym komponentem Material 3: ma własną geometrię i motion wynikające z funkcji produktu, ale nie jest osobnym design systemem ani kopią Cupertino UI.

## 2. Zasady nadrzędne

1. Material 3 jest źródłem bazowych zasad UI/UX.
2. `MaterialTheme` jest jedynym centralnym theme dla ekranów Compose.
3. Standardowe komponenty Material 3 mają pierwszeństwo, jeśli realizują wymagane zachowanie.
4. Custom components są dozwolone tam, gdzie wymaga tego Dynamic Island, call controls, cutout geometry lub morphing.
5. Custom components muszą korzystać z tych samych color/type/shape/spacing/motion tokens.
6. Feature modules nie mogą tworzyć własnego theme ani kopiować tokenów.
7. Nie hardcodujemy przypadkowych wartości wizualnych bez uzasadnionego wpisu do design systemu.
8. Accessibility jest elementem design systemu od pierwszej implementacji.

## 3. Theme architecture

Docelowo design system powinien wystawiać m.in.:

```text
DynamicIslandTheme
├── MaterialTheme.colorScheme
├── MaterialTheme.typography
├── MaterialTheme.shapes
├── IslandSpacing
├── IslandSizing
├── IslandMotion
├── IslandElevation
└── IslandSemanticColors
```

Ekrany Compose używają `MaterialTheme` bezpośrednio.

Overlay renderer, jeśli zostanie wykonany poza Compose, otrzymuje zmapowany zestaw tych samych tokenów.

## 4. Color system

### Ekrany aplikacji

Używać semantic roles Material 3 zamiast kolorów związanych z konkretnym ekranem:

- primary / onPrimary
- secondary / onSecondary
- tertiary / onTertiary
- surface / onSurface
- surfaceVariant / onSurfaceVariant
- error / onError
- outline / outlineVariant

### Dynamic color

Dynamic color może być dostępny jako opcja dla ekranów aplikacji na wspieranych urządzeniach.

Nie może automatycznie zmieniać krytycznych kolorów semantycznych call controls ani pogarszać kontrastu wyspy.

### Dynamic Island surface

Domyślny background wyspy powinien być czarny lub bardzo ciemny, aby wizualnie scalać się z fizycznym otworem kamery.

To jest **kontrolowany token powierzchni wyspy**, a nie lokalnie wpisane `Color.Black` w wielu komponentach.

Przykładowe role domenowe:

- `islandSurface`
- `onIslandSurface`
- `islandSurfaceMuted`
- `callPositive`
- `callDestructive`
- `callActive`
- `mediaAccent`
- `navigationAccent`

Role domenowe powinny mapować się na semantykę Material 3 i być centralnie definiowane.

## 5. Typography

Używać Material 3 typography scale.

Nie tworzyć osobnych font sizes dla każdego feature'u.

Przykładowe mapowanie:

- ekranowe nagłówki → `headline*` / `title*`
- sekcje ustawień → `titleMedium` / `titleSmall`
- wartości i nazwy w expanded island → `titleSmall` / `bodyMedium`
- sekundarne informacje → `bodySmall` / `labelMedium`
- call timer / ETA / countdown → centralny token oparty o Material typography

Dopuszczalne jest dopasowanie rozmiarów dla bardzo małej powierzchni compact island, ale musi być to nazwany token design systemu, nie wartość lokalna.

## 6. Shapes

Standardowe ekrany korzystają z Material 3 shape system.

Dynamic Island wymaga własnych, silnie zaokrąglonych kształtów. Definiujemy je jako rozszerzenie wspólnego shape systemu:

- `islandCompactShape`
- `islandExpandedShape`
- `islandSplitShape`
- `islandTransientShape`

Corner radius podczas morphingu jest animowany pomiędzy tokenami docelowymi.

Nie wpisujemy radiusów bezpośrednio w feature UI.

## 7. Spacing i sizing

Wspólne tokeny powinny pokrywać:

- spacing pomiędzy ikoną a tekstem
- padding compact island
- padding expanded island
- odstępy call controls
- minimalne touch targets
- artwork sizes
- icon sizes
- compact/expanded min/max dimensions

Geometria związana bezpośrednio z cutoutem może być dynamiczna, ale bazowe paddingi i relacje wizualne nadal pochodzą z design systemu.

## 8. Ikonografia

Preferować Material Symbols / oficjalne ikony Material dla standardowych akcji:

- call
- call end
- mic / mic off
- volume / speaker
- Bluetooth
- keypad
- pause / play
- previous / next
- timer
- navigation
- battery / charging

Nie mieszać przypadkowych zestawów ikon.

Custom icons tylko wtedy, gdy brak właściwego odpowiednika.

## 9. Material 3 components

Na zwykłych ekranach preferować odpowiednie komponenty Material 3, m.in.:

- `Scaffold`
- `TopAppBar`
- `NavigationBar` / `NavigationRail`, jeśli potrzebne
- `ListItem`
- `Switch`
- `Slider`
- `Button` / `FilledTonalButton` / `OutlinedButton`
- `IconButton`
- `Card`
- `AlertDialog`
- `ModalBottomSheet`
- `TextField`
- `Snackbar`
- Material progress indicators

Nie budować własnego switcha, buttona czy dialogu tylko dla wyglądu.

## 10. Dynamic Island components

Custom components mogą obejmować:

- `IslandSurface`
- `CompactIsland`
- `ExpandedIsland`
- `SplitIsland`
- `IncomingCallIsland`
- `ActiveCallIsland`
- `MediaIsland`
- `TimerIsland`
- `NavigationIsland`
- `IslandActionButton`

Każdy z nich pobiera tokeny z design-system zamiast definiować własne style.

## 11. Telefonowanie

Call UI ma być semantycznie i wizualnie spójny w trzech miejscach:

1. compact island
2. expanded island
3. opcjonalny full-screen call UI

Ta sama akcja powinna zachowywać tę samą ikonę, rolę koloru i semantics.

Przykład:

- accept → pozytywna semantic action
- hang up / reject → destructive semantic action
- mute → toggle state
- speaker/Bluetooth → selected/unselected state

Nie wolno opierać znaczenia wyłącznie na kolorze.

## 12. Motion

Bazą są zasady ruchu Material 3, ale Dynamic Island potrzebuje rozszerzonego morphingu.

Centralne motion tokens powinny definiować:

- standard transition duration
- fast feedback duration
- expand/collapse spring
- activity replacement spring
- split transition spring
- content fade timing
- scale ranges
- gesture settle behavior

Wymagania:

- animacje przerywalne
- brak layout jump
- continuity zamiast replace/fade
- velocity-aware gesture settle, jeśli implementowane
- brak mnożenia lokalnych `spring()` i `tween()` z różnymi parametrami po feature modules

## 13. State layers i interaction feedback

Każda interaktywna kontrolka musi mieć czytelne stany:

- enabled
- pressed
- focused, jeśli dotyczy
- selected
- disabled
- loading, jeśli dotyczy

Overlay nie może wyglądać jak martwa grafika. Tap/long press/swipe muszą mieć odpowiedni feedback bez psucia płynności morphingu.

## 14. Accessibility

Minimum:

- poprawne Compose semantics
- content descriptions dla ikon, jeśli nie wynikają jednoznacznie z otaczającego tekstu
- właściwe role button/toggle
- czytelny selected state
- wspieranie font scale tam, gdzie powierzchnia pozwala
- brak krytycznych informacji przekazywanych wyłącznie kolorem
- odpowiedni kontrast
- sensowne touch targets dla Expanded UI
- compact island nie może oferować zbyt małych, trudnych do trafienia kontrolek; w takim przypadku akcja prowadzi do expanded state

## 15. Light / dark

Ekrany aplikacji:

- pełny light theme
- pełny dark theme
- opcjonalnie system default

Dynamic Island:

- bazowo dark surface niezależnie od theme ekranu
- treść i akcenty nadal korzystają z semantic roles design systemu

## 16. Adaptive UI

Settings, onboarding i dialer powinny poprawnie działać dla:

- różnych density
- różnych font scale
- portrait
- landscape tam, gdzie system może go wymusić
- foldables / większych ekranów w rozsądnym zakresie

Sama wyspa jest przede wszystkim top-center overlayem zależnym od cutout geometry.

## 17. Zakazane wzorce

Nie implementować:

- osobnego stylu „iOS/Cupertino” dla wyspy
- losowych hex colors w feature modules
- lokalnych font sizes i radiusów bez tokenu
- osobnych icon packs per feature
- custom standard controls bez potrzeby
- abrupt view swap zamiast morphingu
- UI bez dark/light/accessibility consideration
- komponentów zależnych bezpośrednio od konkretnego modelu telefonu, jeśli geometria może pochodzić z cutout data

## 18. Definition of Done dla tasku UI

Każdy task dotykający UI jest ukończony dopiero, gdy:

1. używa wspólnego Material 3 theme,
2. nie wprowadza lokalnego design systemu,
3. kolory są semantic/token-based,
4. typography pochodzi z tokenów,
5. spacing/shapes używają wspólnego systemu,
6. interakcje mają poprawne stany,
7. accessibility semantics są dodane,
8. light/dark zachowują czytelność, jeśli ekran je obsługuje,
9. motion korzysta z centralnych tokenów,
10. custom UI jest uzasadnione funkcją, nie samą chęcią odmiennego wyglądu.
