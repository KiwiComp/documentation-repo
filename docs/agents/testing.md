# Testing

## Principles

- Code and tests support each other. Code that is hard to test is a signal that its design should be reviewed.
- Tests do not govern the code. Do not add anything that only tests need, such as test keys, hooks or widened visibility, if it makes the code harder to read or worse. Change the test approach instead.

**The rule:** a test must be able to fail when the behavior it tests breaks. Therefore, when a test performs an action, verify that the starting state differs from the expected result, perform the action, then verify the result.

## What is tested, and where

Every new feature has tests for the code below.

| Code | Source set | How |
|---|---|---|
| ViewModel | `src/test` | Fakes for repositories and app-wide services |
| Use case | `src/test` | Fakes for repositories |
| Repository with logic | `src/test`, or `src/androidTest` if a data source has no interface | Fakes for the interfaces of its data sources; a data source without an interface uses in-memory Room |
| DAO with its own queries | `src/androidTest` | In-memory Room |
| Room migration | `src/androidTest` | `MigrationTestHelper`, for every migration |
| Every Screen | `src/androidTest` | `createComposeRule`, with data passed as parameters |
| Shared composable in `core.ui.components` | `src/androidTest` | `createComposeRule`, with data passed as parameters |
| `Navigator` | `src/test` | Call its functions and check the resulting back stack |
| Network model with custom names, defaults or serializers | `src/test` | Decode sample JSON with the app's `Json` configuration |

## Tools

JVM tests use JUnit 4, `kotlinx-coroutines-test`, Turbine and assertions from `kotlin.test`.

## Rules

- Screen tests cover every state the Screen can show, and check that events reach their lambdas.
- Screen tests do not include the Route or the ViewModel, so `src/androidTest` needs no fakes.
- Routes have no automated tests, so keep logic out of them. List what a new or changed Route does (navigation, messages, dialogs) under what the developer should test manually.
- Fakes live in `src/test` and are named after the interface (`FakeItemRepository`). Do not use mocking libraries.
- Inject test versions of dispatchers and `Clock`. Set `Dispatchers.Main` with a shared JUnit rule.
- Test `Flow` with Turbine. `stateIn` uses `WhileSubscribed`, so a test must collect the flow for the state to update.
- In JVM tests, call `Snapshot.sendApplyNotifications()` after changing text field state that a `snapshotFlow` reads. Otherwise the flow does not emit the change.
- Compose tests find text through string resources, so they do not depend on the device's language.
- A bug fix gets a regression test if the bug can be reproduced in a test.
- Tests make no real network calls.
