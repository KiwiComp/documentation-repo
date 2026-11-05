---
name: new-project
description: Set up the app-wide foundation in a new, empty Android project so that features can be built on it. Use when the developer starts a new project.
disable-model-invocation: true
---

# New project

Set up what `docs/agents/architecture-principles.md` describes as the app-wide foundation. Add no
features. The project must already exist, created with Android Studio's Empty Activity template,
in a Git repository with a GitHub remote. The template, `docs/` and `.claude/` must be committed
and pushed to the default branch, so that pull requests have a base. If `gradlew`, the remote or
that commit is missing, stop and ask the developer to add it.

## 1. Read and ask

- Read `docs/agents/working-conventions.md`, `docs/agents/architecture-principles.md` and
  `docs/agents/testing.md`.
- Read the Gradle files, the version catalog and the existing source. Do not create, edit, move or
  delete files in this step or the next two.
- Ask only for what the project does not already answer:
  - which languages the app supports,
  - whether the app has tabs, and which ones,
  - the first screen, as a name only,
  - whether the app uses Room, the network, or both.

## 2. Plan

Share a short plan with these headings:

- **Dependencies**: the libraries and plugins that are added. Never guess a version. If you cannot
  verify one, ask.
- **Files**: the packages and files that are added or changed, from the list under Foundation.
- **Branch**: `chore/project-setup`, following the Git conventions in
  `docs/agents/working-conventions.md`.
- **Open questions**: anything still undecided. Leave the heading out if there is nothing.

## 3. Confirm

- Wait for the developer to explicitly confirm the plan.
- The confirmation is the developer's instruction to add the dependencies and to create and edit
  the files in the plan, and only those, plus files the build generates from them that belong in Git.

## 4. Implement

Run `git status`. If there are uncommitted changes, stop and ask.
Create the branch with `git switch -c chore/project-setup`.

### Foundation

- Build: Hilt, Navigation 3, Compose and the test tools in `docs/agents/testing.md`, in the version
  catalog.
- Root package: the `@HiltAndroidApp` application class, `MainActivity`, the root composable with the
  root `SnackbarHost`, and `AppViewModel`.
- `navigation`: `NavDisplay` and `Navigator`, plus `TopLevelDestination` if the app has tabs.
- `core.common`: the dispatcher qualifiers, `MessageQueue` and its implementation, `UserMessage`, and
  the Hilt module that provides them. Do not add `Clock`: it is added when code first reads the
  current time.
- `core.ui`: the theme, the conversion of `UserMessage` to text, and `strings_common.xml` with its
  translations.
- `feature.<first-screen>`: a Route and a Screen showing only a title, and its navigation file, so
  that the app can start.
- `core.database`, only if the app uses Room: an empty start is not possible, so create the database
  with the first feature's entities. Add only the schema export directory and the build setup here.
- `core.network`, only if the app uses the network: the Hilt module with Retrofit and the app's
  `Json` configuration.
- Tests in `src/test`: the shared JUnit rule for `Dispatchers.Main`, `FakeMessageQueue`, and the tests
  for `Navigator` and the message queue.
- Tests in `src/androidTest`: the tests for the conversion of `UserMessage` to text and the first
  Screen.

Run the checks in the verify skill (`.claude/skills/verify/SKILL.md`). Never commit or push.

## Report

Answer the developer with:

- the summary described under "Definition of done" in `docs/agents/working-conventions.md`,
- the facts to add to the project's own docs: supported languages, whether the app has tabs, and
  whether it uses Room and the network.
