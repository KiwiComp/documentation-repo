# Jetpack Compose App Architecture

Nov 4, 2026 · @Kim

## Architecture principles

The app is a single Gradle module (`app`) layered as displayed below under Layers and dependencies. Dependencies point one way, each layer keeps its own types to itself, and every screen is a stateful Route around a stateless Screen.

### Layers and dependencies

```mermaid
flowchart TD
    root["root package<br/>MainActivity and the root composable"]
    navigation["navigation<br/>NavDisplay, Navigator, top-level destinations"]
    feature["feature<br/>Route, Screen, ViewModel, UiState"]
    domain["core.domain (optional)<br/>Use cases for logic shared between ViewModels"]
    data["core.data<br/>Repositories: the only way to reach app data"]
    database["core.database<br/>Room entities and DAOs"]
    network["core.network<br/>Retrofit, NetworkX models"]
    ui["core.ui<br/>Theme and shared composables"]
    common["core.common<br/>Dispatchers, Clock, message queue, UserMessage"]
    model["core.model<br/>App models as plain data classes"]

    root --> navigation
    navigation --> feature
    feature --> domain
    feature --> data
    domain --> data
    data --> database
    data --> network
    feature --> ui
    domain --> common
    data --> model

    style domain stroke-dasharray: 5 5
```

The diagram shows the main dependencies. The list below is the complete rule: anything not listed is forbidden.

- `core.model` → nothing. Plain data classes only.
- `core.common` → nothing. App-wide types and services: dispatcher qualifiers, `Clock`, the message queue, `UserMessage`.
- `core.database`, `core.network` → nothing.
- `core.data` → `core.model`, `core.common`, `core.database`, `core.network`
- `core.domain` → `core.model`, `core.common`, `core.data`
- `core.ui` → `core.model`, `core.common`. No state, ViewModels or data access.
- `feature` → `core.model`, `core.common`, `core.data`, `core.domain`, `core.ui`. Never another feature.
- `navigation` → `feature`
- Root package (`MainActivity`, root composable) → `navigation`, `core.ui`, `core.common`

Each layer has its own Hilt module in its own `di` package. Bind interfaces with `@Binds`.

The compiler doesn't enforce this in a single module, so check imports in every review.

### Data layer

Repositories are the only way to reach app data, and only app models leave the data layer.

- Each repository is an interface (`ItemRepository`) with a fake for tests and an implementation named after its source (`OfflineFirstItemRepository`, `RoomItemRepository`).
- Repository functions use app language (`addItem`, `observeItems`). Reads that change over time return `Flow` of app models. One-shot reads and writes are `suspend`.
- Each kind of data has an app model, plus one model for each source it comes from:
  - `Item` (plain data class) in `core.model`, always
  - `NetworkItem` (`@Serializable`) in `core.network.model`, if it comes from the network
  - `ItemEntity` (`@Entity`) in `core.database.entity`, if it is stored in Room
- Conversions (`NetworkItem.toItem()`, `ItemEntity.toItem()`, `Item.toEntity()`) are private extension functions in the repository implementation.
- With Room, stored data is read only from Room, its single source of truth. Data the app doesn't store, such as search results, may come straight from the network.
- Repositories let exceptions propagate. The ViewModel decides how to show them.
- Inject dispatchers with a qualifier (`@IoDispatcher`). Never use `Dispatchers.*` directly.
- Room schema changes use `AutoMigration` when possible and a manual `Migration` when needed, such as when data must be transformed. Never use `fallbackToDestructiveMigration`.
- Room uses `exportSchema = true`, and the schema files are committed to Git.

### Domain layer (optional)

Add a use case only when logic is shared between ViewModels or combines several repositories; otherwise the ViewModel calls the repository directly.

- A use case is a class in `core.domain` with one `operator fun invoke`, created with `@Inject constructor`, for example `ObserveActiveItemsUseCase`.
- It depends only on repositories and app-wide services, and returns app models, usually as a `Flow`.

### ViewModels

- ViewModels are `@HiltViewModel` and depend only on repository interfaces, use cases and app-wide services (dispatchers, `Clock`, the message queue).
- Content screens: UI state is one `StateFlow<XUiState>`, built from the repository's `Flow` with `stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), XUiState.Loading)`. Never collect in `init` and copy into a `MutableStateFlow`.
- Forms: UI state, except text field input, is a private `MutableStateFlow(XUiState())`, exposed as `StateFlow`. An edit form loads its initial values with a one-shot `suspend` read, tracked with `isLoading`. If the read fails with an expected exception, the state holds a `loadError: UserMessage?`, shown as a blocking error with retry or back, and `retry()` runs the read again.
- One-time messages are a separate flow (see State and errors below).
- Screen input that the state depends on is merged with `combine` before `stateIn`. Text field input is `mutableStateOf` with a private setter. When the state depends on it, convert it with `snapshotFlow { query }` before `combine`. Other input (selected option, page number) is a private `MutableStateFlow`.
- Screen events are plain functions (`save()`, `delete(id)`). Writes run in `viewModelScope.launch`.
- Navigation arguments arrive as ids and other simple values through assisted injection, never as the destination or via `SavedStateHandle`, so tests run on the JVM.
- ViewModels never navigate. When navigation follows data work, the state says so and the Route navigates: a flag in a form's data class (`isSaved`), or a state of its own on a content screen (`XUiState.Deleted`), combined in before `stateIn` so it wins over the item disappearing from the `Flow`.

### UI: Route and Screen

Each `XScreen.kt` holds two composables, `XRoute` and `XScreen`.

- Route: gets its ViewModel with `hiltViewModel()`, or with `hiltViewModel<XViewModel, XViewModel.Factory> { it.create(id) }` when it has navigation arguments, and collects state with `collectAsStateWithLifecycle()`. It owns UI-only state, such as dialog visibility (`rememberSaveable`) and the `SnackbarHostState`, and reacts to navigation flags or states and to messages with `LaunchedEffect`.
- Screen: data comes in as parameters and events go out as lambdas. It never receives a ViewModel, the `Navigator` or a destination. It may `remember` only UI element state such as scroll position. Anything that decides what is shown, including whether a dialog is open, is a parameter.
- Lambdas describe what the user did, not where to go: `onOpenItem`, `onSaved`, `onBack`.
- Preview the Screen in every state (loading, error, empty, content, open dialogs), in light and dark mode.
- Composables stay in the feature that uses them. Only those shared by several features go in `core.ui.components`.
- User-facing strings live in one resource file per feature (`res/values/strings_profile.xml`). Strings shared by several features, and those used by `core.ui`, go in `strings_common.xml`.
- String names start with the feature (`profile_title`), or with `common_` in the shared file (`common_save`).
- Share a string only when the meaning is the same. The same word in different contexts gets separate strings, since translations may differ.
- English is the default language in `values/`, with translations in folders such as `values-sv/`. The project's own docs state which languages the app supports.

### Navigation

- Use Navigation 3 (`androidx.navigation3`). Destinations are `@Serializable` classes implementing `NavKey`, named `XDestination`. They carry only ids and other simple types (`String`, `Long`, `Int`, `Boolean`).
- Each feature has `<Name>Navigation.kt` in `feature.<name>.navigation`, holding its destinations and an entry builder `EntryProviderScope<NavKey>.xEntry(...)`. The entry builder calls `XRoute` with the destination's values and the screen's event lambdas.
- The app's `navigation` package holds `NavDisplay` and `Navigator`, plus `TopLevelDestination` if the app has tabs. Destinations are referenced only there and in their own feature's navigation file. Only `NavDisplay` connects one feature's lambdas to another feature's destinations.
- Only `Navigator` changes the back stack. It is the app's own class, not part of Navigation 3. Back stack rules, such as replacing the current screen, are `Navigator` functions, never list operations elsewhere. With tabs, it keeps one back stack per `TopLevelDestination`.
- `NavDisplay` uses `rememberSaveableStateHolderNavEntryDecorator()` and `rememberViewModelStoreNavEntryDecorator()`, so each entry gets its own ViewModel.

### State and errors

- Screens that load content use a sealed `XUiState`: `Loading`, `Success`, `Error(message: UserMessage)`, plus any state that triggers navigation (`Deleted`). Empty is `Success` with an empty list. Forms use a data class with flags such as `isLoading` and `isSaving`.
- Derived values are getters, not stored fields. UiState never holds a `Throwable`, an exception message or user-facing text.
- Errors are either:
  - Blocking: content couldn't be loaded. Turn the flow's expected exceptions into `Error` with `.catch` before `stateIn`, and show it with retry or back.
  - Transient: an action failed while content is shown. Keep the content and show a snackbar through a one-time message.
- On content screens, retry restarts the content flow: build it inside `retryTrigger.flatMapLatest { … }`, with `.catch` and `onStart { emit(XUiState.Loading) }` inside, where `retryTrigger` is a private `MutableStateFlow<Int>` that `retry()` increments.
- Catch only expected exceptions (`IOException`, `HttpException`, `android.database.SQLException`). Never catch `Exception` or `Throwable`, and never wrap suspend calls in `runCatching`. These exception types are a deliberate exception to layers keeping their types to themselves: a ViewModel may import them, and nothing else from the libraries behind the data layer.
- One-time messages are a `UserMessage` enum in a separate `StateFlow<UserMessage?>`, backed by a private `MutableStateFlow`. The Route shows it and calls `onMessageShown()`. `core.ui` turns `UserMessage` into text.
- Messages that must outlive their screen, such as "not found" after leaving it, go through the app-wide message queue, shown by the root `SnackbarHost`.
