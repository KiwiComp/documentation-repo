Follow these working conventions in every task. "The project docs" means the project's own instruction and documentation files, such as AGENTS.md, CLAUDE.md, README, CONTRIBUTING and any plan, architecture, testing or other agent docs (for example under docs/), but not this file. Where the project docs set a different convention than this file, the project docs win.

## Language

- All code, identifiers, comments, doc comments, commit messages and branch names are in English.

## Before writing code

- Read the existing code in the area you are changing, and follow its patterns.
- If existing code differs from the project docs, point out the difference. Do not silently follow either one, and do not rewrite unrelated code to make it match.
- For a new feature, or a change that touches several files, start with a short plan: which files you will create or change, and in what order. Wait for the developer's approval before writing code.
- For a small fix in one place, go ahead without a plan.

## Scope

- One feature per branch. If a feature is large, split it into steps that each leave the project building with all tests passing.
- Change only what the task requires. If you notice something else that should be fixed, mention it in your summary instead of changing it.

## Ask before

- Adding a dependency, plugin or tool that the project does not already use and the project docs do not list.
- Deviating from any of the project docs.
- Changing a database schema or a stored data format (it usually requires a migration).
- Deleting files.
- Making a change that affects several features at once.

Decide on your own anything the project docs or the existing code already answer, such as names, file placement and how a test is written.

## Git

- Create a branch for each task from an up-to-date default branch (usually main), named after what it adds: feat/<name>, fix/<name>, refactor/<name>, chore/<name>, build/<name>, ci/<name>, test/<name> or docs/<name>.
- Never commit without asking the developer first. Leave changes uncommitted so the developer can review the diff. Committing, pushing and opening a pull request are done through the /pull-request skill.
- Commit messages and pull request titles use `<type>: <description>`, where the type is the same as the branch prefix: `feat/user-profile` → `feat: add user profile`.
- The description is in the imperative mood, starts in lowercase, has no period at the end and keeps the whole first line under 72 characters.
- Never push, merge, rebase onto the default branch, or force push without asking the developer first.
- Pull requests are merged with squash and merge, so the pull request title becomes the commit message on the default branch.
- After a merge, start new work on a new branch from an up-to-date default branch. Never continue on a merged branch.

## Explaining choices

- When you make a choice that does not follow directly from the project docs, explain it briefly: what you chose and why.
- Do not explain what the project docs already state.

## Definition of done

A task is done when:

- the checks in the /verify skill pass, except those that could not be run,
- the code and its tests follow the project docs,
- and you have written a summary.

The summary contains:

- what was changed, file by file at a high level,
- the choices you made that were not given by the project docs,
- what the developer should test manually,
- anything you could not verify (for example tests that need a device, an emulator or an external service),
- anything you noticed outside the task that should be addressed.

## When unsure

If a requirement is unclear, a rule seems wrong for the case at hand, or you cannot verify something (a library version, an API, a behavior), say so and ask. Never guess and present the guess as fact.
