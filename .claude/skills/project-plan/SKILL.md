---
name: project-plan
description: Plan a new app's features, screens and data models by asking the developer, and write the result to docs/project-plan.md. Use when the developer starts a new project, before /new-project.
disable-model-invocation: true
---

# Project plan

Find out what the app should contain and write it down, so that /new-project and /new-feature can
start from it. Write no code.

The project must already exist, created with Android Studio's Empty Activity template, in a Git
repository with a GitHub remote. The template, `docs/` and `.claude/` must be committed and pushed
to the default branch. If any of this is missing, stop and ask the developer to add it.

If `docs/project-plan.md` already exists, stop and ask the developer whether to change it.

## 1. Ask

- Read `docs/agents/working-conventions.md`.
- Do not create, edit, move or delete files in this step or the next two.
- Ask a few questions at a time, and offer options when there is an obvious choice:
  1. **App**: what it does, who it is for and what problem it solves.
  2. **Features**: what is included in the first version, what can wait and what is out of scope.
     Suggest an order to build the first version in, and ask the developer to confirm it.
  3. **Screens**: which screens there are, how the user moves between them and whether the app has
     tabs.
  4. **Data models**: which things the app handles, their fields and how they relate, and whether
     each is stored locally, comes from an API, or both.
  5. **Languages**: which languages the app supports.
- If the developer does not know an answer, add it to Open questions and move on.

## 2. Draft

Share the whole draft, following the format below. Keep to these limits:

- No code, class names or tables. Describe data models in plain language.
- One or two sentences per feature. /new-feature clarifies the details.

```markdown
# Project plan

The code is the truth for what is built. This plan shows what is intended.

## App
What the app does, who it is for and what problem it solves.

## Features
In build order:
1. <Name> (planned): one or two sentences.

## Later

## Out of scope

## Screens

## Data models

## Languages

## Open questions
```

Leave out the headings Later, Out of scope and Open questions if there is nothing under them.

## 3. Confirm

- Wait for the developer to explicitly confirm the draft. If they want changes, update it and share
  it again.
- Do not review the draft again on your own, and do not look for further questions. Anything still
  unclear stays under Open questions.

## 4. Write

- Run `git status`. If there are uncommitted changes, stop and ask what to do with them.
- Find the default branch with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`.
  Switch to it with `git switch <default-branch>`, then run `git pull --ff-only`. If the pull fails,
  stop and report it.
- Run `git switch -c docs/project-plan`.
- Write the confirmed draft to `docs/project-plan.md`. Never commit or push.

## Report

Tell the developer the next steps:

1. Run /pull-request and merge the pull request.
2. Run /new-project from the updated default branch.
