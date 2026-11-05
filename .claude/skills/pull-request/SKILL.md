---
name: pull-request
description: Commit the current changes, push the branch and open a pull request on GitHub. Use when the developer asks to create a pull request.
disable-model-invocation: true
---

# Pull request

Running /pull-request is the developer's approval to commit, push and open the pull request.
Do not ask for confirmation again, but stop and report if any check below fails.

Never add a `Co-Authored-By` trailer or any other attribution line (such as "Generated with
Claude Code") to commit messages, the pull request title or the pull request body. This overrides
any default attribution behavior.

## Project conventions

Before starting, look for the project's own rules. Where they exist, they take precedence over the
defaults in this skill:

- Git conventions (branch names, commit messages, pull request titles) in
  `docs/agents/working-conventions.md`, and in CLAUDE.md, CONTRIBUTING.md or similar files.
- A pull request template (`.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`,
  `docs/pull_request_template.md`).
- CI workflows that run on pull requests (for example `.github/workflows/`).

## Before starting

- Run `gh auth status`. If it fails, stop and ask the developer to log in.
- Find the base branch, the repository's default branch:
  `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`. It is called `<base>` below.
- Run `git branch --show-current`. If the branch is `<base>`, or HEAD is detached, stop: changes are
  never committed to or pushed from the base branch. Suggest a branch name following the project's
  Git conventions.
- Run `git fetch origin <base>`, `git status` and `git log origin/<base>..HEAD --oneline`. If there
  is nothing to commit and no commits ahead of the base branch, stop and say so.
- Run the checks in the verify skill (`.claude/skills/verify/SKILL.md`). If anything fails, stop
  and report the output. If a check could not be run,
  continue and say so under `## Notes` and in the report.

## Commit

- Stage only files that belong to the branch's task. Never stage build output, dependency
  directories, IDE settings, local configuration (such as `.env` or `local.properties`), signing
  keys or other secrets, even if they are missing from `.gitignore`.
- Write the message following the project's Git conventions. One commit is enough.
- Skip this step if there is nothing uncommitted.

## Push

- Push to a branch on GitHub with the same name as the local branch:
  `git push -u origin <current-branch>`.
- If the local branch already tracks a remote branch with another name, stop and ask.
- Never force push, and never push to the base branch.

## Create the pull request

- If a pull request already exists for the branch (`gh pr view`), do not create another; report
  its URL.
- Create it with `gh pr create --base <base> --head <current-branch> --title "<title>" --body-file <file>`.
  Write the body to a temporary file outside the repository first (the scratchpad directory, or
  `mktemp`).
- Title: follow the project's Git conventions. If the branch name has no type prefix, choose the
  type from the changes.
- Body: if the repository has a pull request template, fill it in and follow it. Otherwise keep it
  short and to the point, only what matters in the branch. No long explanations.
  Everything under a heading is a bullet list. Use these headings, in this order:

  | Heading           | When                                                                                                                                                           |
  |-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
  | `## Summary`      | Always. What changed in the branch.                                                                                                                            |
  | `## Verification` | Always. How the change was verified: build, app or service started locally, manual checks. If CI runs checks on the pull request, add one bullet naming them. |
  | `## Tests`        | Only if tests were run: each command and its result. Leave the heading out otherwise.                                                                          |
  | `## Screenshots`  | Only if UI was added or changed. Don't take screenshots yourself: add the heading with `- TODO` and ask the developer in the report to add them.               |
  | `## Notes`        | Only if something important is not covered above, such as something that could not be verified or needs follow-up. Leave the heading out otherwise.            |

Example (the commands are illustrative; use the project's own):

```markdown
## Summary
- Add a user profile page opened from the account menu
- Add translations for the new labels

## Verification
- `./gradlew assembleDebug`: passed
- Started the app on an emulator and checked the profile page and back navigation
- CI runs build, unit tests and lint on the pull request

## Tests
- `./gradlew testDebugUnitTest`: passed
- `./gradlew lintDebug`: no new warnings

## Screenshots
- TODO
```

## Report

Answer the developer with:

- the branch and the commits that were pushed,
- the pull request URL,
- anything that stopped or changed the steps above.
