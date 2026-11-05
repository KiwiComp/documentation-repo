---
name: new-feature
description: Start work on a new feature. Ask the developer what it is, clarify until it is clear, share a plan, and implement it once the developer confirms. Use when the developer wants to begin something new.
disable-model-invocation: true
argument-hint: [short description of the feature]
---

# New feature

The developer chooses what to build next. If `docs/project-plan.md` exists, start from it, but find
out what the developer wants now and do not assume that the plan's order still applies.

## 1. Ask

- If the developer described the feature when running /new-feature ($ARGUMENTS), start from that.
  Otherwise ask what the feature is about.
- Do not create, edit, move or delete files in this step or the next two.

## 2. Clarify

- Read `docs/agents/working-conventions.md`, `docs/agents/architecture-principles.md`,
  `docs/agents/testing.md` and, if it exists, `docs/project-plan.md` before planning.
- If the app-wide foundation in `architecture-principles.md` is missing from the code, stop and
  suggest that the developer runs /new-project first.
- Read the code the feature touches before asking, so questions are about real gaps, not about
  things the code already answers.
- The feature is clear when you know:
    - what the user sees and does, including each screen's states (such as loading, empty and error),
    - what data is stored or changed,
    - what is out of scope.
- If anything is unclear, ask follow-up questions. Ask a few specific ones at a time, and offer
  options when there is an obvious choice. Repeat until it is clear.
- Do not fill gaps with your own assumptions. If something is a matter of taste, ask.

## 3. Plan

Share the plan. Keep it short. Use these headings:

- **Goal**: what the feature does for the user, in one or two sentences.
- **Scope**: what is included and what is not.
- **Steps**: the changes in order, per layer or module as architecture-principles.md divides the
  code (for example data, domain, UI, navigation), naming the packages and files that are added,
  changed or deleted. Follow the project docs (such as CLAUDE.md or AGENTS.md) and
  architecture-principles.md.
- **Strings**: new user-facing strings, in every language the app supports. Leave the heading out
  if there are none.
- **Dependencies**: new libraries, plugins or tools, and why they are needed. Leave the heading out
  if there are none.
- **Database migrations**: Room schema changes and how existing data is migrated. Leave the heading
  out if there are none.
- **Project docs**: changes to `AGENTS.md` and `docs/project-plan.md`, such as changed project facts,
  new lasting deviations from `docs/agents/`, or marking the feature as done in the plan. Leave the
  heading out if there are none.
- **Tests**: the tests that are added or changed, as `testing.md` requires.
- **Branch**: a branch name following the Git conventions in `docs/agents/working-conventions.md`.
  If the feature is split into steps, the plan covers the first step, and the later steps are listed
  under Scope.
- **Open questions**: anything still undecided. Leave the heading out if there is nothing.

## 4. Confirm

- Wait for the developer to explicitly confirm the plan. Do not start on anything else.
- If the developer wants changes, update the plan and share it again.
- The confirmation is the developer's instruction to create, edit, move and delete the files in
  the plan, and only those, plus files the build generates from them that belong in Git, such as
  Room schema files.

## 5. Implement

- Before changing any files:
  - Run `git status`. If there are uncommitted changes, stop and ask what to do with them.
  - Find the default branch with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`;
    if that fails, it is usually `main`. Switch to it with `git switch <default-branch>`, then run
    `git pull --ff-only`. If the pull fails, stop and report it.
  - `git switch -c <branch>` with the branch name from the plan.
- Implement the plan step by step.
- If the plan turns out to need a change (a missing file, a different approach), stop, explain it
  and wait for the developer's confirmation before continuing.
- Run the checks in the verify skill (`.claude/skills/verify/SKILL.md`).
- Never commit or push.

## Report

Answer the developer with the summary described under "Definition of done" in
`docs/agents/working-conventions.md`, and add any deviation from the plan.
