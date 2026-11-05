---
name: verify
description: Build the Android app and its instrumented tests, run the unit tests and lint, check layer imports and, when needed, run the instrumented tests, then report the results. Use after every code change, before a pull request, or when the developer asks to verify the project.
---

# Verify

Run the checks below from the project root, in this order (use `gradlew.bat` instead of
`./gradlew` on Windows). Stop at the first failure and report it. Never weaken, ignore or skip a
test to make a check pass.

If the project docs (such as AGENTS.md, CLAUDE.md or docs/) name other tasks or extra checks,
follow them. If the project uses product flavors, use the flavor's variant tasks instead, for
example `assembleDevDebug` and `lintDevDebug`; `./gradlew tasks` lists them.

## Checks

1. `./gradlew assembleDebug`
2. `./gradlew assembleDebugAndroidTest`, which compiles the instrumented tests without a device
3. `./gradlew testDebugUnitTest`
4. `./gradlew lintDebug`
5. Static analysis, only if the project configures it (for example detekt, ktlint or Spotless in the
   Gradle build files): run its check task, such as `./gradlew detekt`, `./gradlew ktlintCheck` or
   `./gradlew spotlessCheck`. Never run a task that rewrites files, such as `spotlessApply` or
   `ktlintFormat`, unless the developer asks.
6. Layer imports: compare the imports in the changed Kotlin files under `src/main/` (find them as in
   step 7) with the import rules in `docs/agents/architecture-principles.md`. Each forbidden import
   is a failure.
7. `./gradlew connectedDebugAndroidTest`, only when androidTest code or the code it covers changed:
   - Find the changed files with `git diff --name-only <default-branch>...HEAD` and
     `git diff --name-only HEAD`, plus untracked files from `git status --porcelain`. Find the
     default branch with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`; if that
     fails, it is usually `main`. If the repository has no commits yet, treat all files as changed.
   - Instrumented tests are needed when a file under a `src/androidTest/` directory changed, or
     when a changed main file is covered by instrumented tests. A main file is covered when:
     - `docs/agents/testing.md` or the other project docs say that this kind of code is tested on a
       device, or
     - its classes or composables are referenced from a file under `src/androidTest/` (search the
       androidTest sources for their names).
   - Run `adb devices`. If no device or emulator is listed with the state `device` (not
     `unauthorized` or `offline`), do not run the task and say so explicitly. Never claim that
     instrumented tests pass when they were not run.
   - Compose UI tests find text through string resources (see `docs/agents/testing.md`), so they do
     not depend on the device's system language. If a test fails because it has hard-coded text that does not
     match the device's language, report that the test breaks this rule, not that the app has a bug.

## When a check fails

- Show the part of the output that explains the failure: the compiler error, the failing test
  with its assertion message, the lint errors, or the file and the forbidden import. Not the whole
  log.
- During a task where you changed the code, including /new-feature, fix failures caused by your own
  change and run the checks again. For a failure that was there before the task, follow "Definition
  of done" in `docs/agents/working-conventions.md`.
- Otherwise, including during /pull-request or when the developer only asked you to verify, do not
  start fixing anything unless the developer asked for it.

## Report

Answer the developer with:

- each task and check that was run and its result (passed, failed, number of tests if Gradle prints
  it),
- each task and check that was not run, and why,
- lint and static analysis warnings, only those in files changed on the branch.
