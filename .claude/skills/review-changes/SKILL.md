---
name: review-changes
description: Review the changes on the current branch for important bugs and breaks of the project's rules, and report only what would stop a merge. Use when the developer asks for a review.
disable-model-invocation: true
---

# Review changes

Review the changes on the current branch and report the important problems. Do not edit, commit or
push anything, and do not run Gradle: the build, the tests and lint belong to the verify skill.

## 1. Read

- Find the changed files with `git diff --name-only <default-branch>...HEAD` and
  `git diff --name-only HEAD`, plus untracked files from `git status --porcelain`. Find the default
  branch with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`; if that fails, it
  is usually `main`.
- If nothing has changed, say so and stop.
- Read `docs/agents/architecture-principles.md`, `docs/agents/testing.md` and `AGENTS.md`.
- Read the changes, and as much of the surrounding code as you need to understand them.

## 2. Decide what to report

Report a problem only if the answer to both questions is yes:

1. **Can you point to it?** It breaks a specific rule in the documents above, or there is a concrete
   scenario in which the app does the wrong thing. Otherwise it is a matter of taste.
2. **Is it important?** A developer would stop the merge because of it.

Report, for example:

- Bugs: crashes, wrong behavior, lost data or user input, leaked secrets.
- Breaks of the architecture, such as a forbidden layer dependency, a ViewModel that navigates, a
  Screen that receives a ViewModel, direct use of `Dispatchers.*`, `catch (e: Exception)`,
  `fallbackToDestructiveMigration`, a Room change without a migration, or hard-coded user-facing
  text.
- Tests that `docs/agents/testing.md` requires but that are missing, and tests that cannot fail.

Do not report:

- Style, formatting or wording.
- An alternative solution that is only equally good.
- Refactorings, small performance improvements or comments.
- Small deviations without practical consequences, for example in a name or a file's placement.
- Problems in code that the changes do not touch.
- Deviations listed under Approved deviations in `AGENTS.md`.

If you are unsure whether a problem is real, read more of the code. If you are still unsure, leave
it out, unless it could cause a crash or data loss. Then report it and mark it as unverified.

## 3. Report

Make one pass, and report the most serious problem first. For each problem, give:

- the file and line,
- the problem,
- the rule it breaks, or the scenario in which it fails,
- a short suggested fix.

If there are no problems to report, answer "No important problems found". Do not add praise, and do
not add smaller findings to fill the report.

Do not fix anything until the developer chooses what to fix.
