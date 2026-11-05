# [App name: not set]

[About: not set. What the app does and who it is for.]

## Features

[Not set. The features that are built. Planned features are in `docs/project-plan.md`.]

## Getting started

1. Clone the repository and open it in Android Studio.
2. [Not set. Any setup this app needs, such as API keys. Write "None" if there is nothing.]
3. Run the `app` configuration.

---

## Working with agents

This project is built together with AI agents. The documents and skills below tell them how to work, so that every project starts from the same base.

You need Android Studio, the GitHub CLI (`gh`) and an agent that can read these files. Instrumented tests also need a device or an emulator.

### The developer is in control

The agent does not control the project. You decide what is built, what is accepted and what is merged. The agent proposes and implements, and every step below waits for you.

1. **The agent proposes a plan.** It lists what it will change and in what order. It writes no code before you approve.
2. **You approve or reject the plan.** You can also ask for changes, and the agent then shares an updated plan. Nothing starts until you explicitly approve.
3. **The agent implements the approved plan.** It does only what the plan says. If the plan turns out to need a change, the agent stops and waits for your approval. When it is done, it runs `/verify` and writes a summary.
4. **You review the diff.** The agent may not commit or push, so its changes stay uncommitted. Read the diff, test what the summary lists and ask for changes if needed. You can run `/review-changes` for support, but it does not replace your own review.
5. **You decide when the work moves on.** Only you can start `/pull-request`, which is your approval for the agent to commit, push and open a pull request. You review and merge the pull request.

A small fix in one place needs no plan, so it starts at step 3. Steps 4 and 5 always apply.

The agent also asks you before it adds a dependency, deviates from the project docs, changes a database schema or a stored data format, deletes files, or makes a change that affects several features. An approved plan counts as your answer for what it lists. If the agent is unsure, it asks instead of guessing.

### Documents

| File | Content |
|---|---|
| `AGENTS.md` | The agents' starting point: what to read, the skills, project facts and approved deviations |
| `CLAUDE.md` | Makes Claude Code read `AGENTS.md` |
| `docs/agents/working-conventions.md` | How agents work: plans, scope, Git and definition of done |
| `docs/agents/architecture-principles.md` | Layers, data, ViewModels, UI, navigation and errors |
| `docs/agents/testing.md` | What is tested, where and how |
| `docs/project-plan.md` | The app's intended features, screens and data models, created by `/project-plan` |

### Skills

| Skill | What it does | Started by |
|---|---|---|
| `/project-plan` | Asks about the app and writes `docs/project-plan.md` | Developer |
| `/new-project` | Sets up the app-wide foundation and fills in the project-specific parts of `AGENTS.md` and this file. Stops if the project plan is missing | Developer |
| `/new-feature` | Clarifies a feature, shares a plan and implements it once you confirm | Developer |
| `/verify` | Builds the app and runs tests, lint and the layer import check | Agent or developer |
| `/review-changes` | Reviews the branch's changes and reports only important bugs and breaks of the project's rules. Changes nothing | Developer |
| `/pull-request` | Commits, pushes and opens a pull request | Developer |

### Start a new project

1. Create the project in Android Studio with the Empty Activity template.
2. Run this command in the project's root to get the base from its `main` branch. It overwrites files with the same name, such as the template's `.gitignore`:

   ```bash
   gh api repos/KiwiComp/documentation-repo/tarball | tar -xz --strip-components=1
   ```

3. Create the GitHub repository, then commit and push everything to the default branch.
4. Run `/project-plan`, then `/pull-request`, and merge the pull request.
5. Run `/new-project`, then `/pull-request`, and merge the pull request.

### Build a feature

1. Run `/new-feature`, answer its questions and confirm the plan.
2. Review the changes and test what the agent's summary lists. Run `/review-changes` if you want support.
3. Run `/pull-request` and merge the pull request.

For a small fix, describe it to the agent directly and run `/pull-request` when you are happy with it.

### Project-specific content

Agents ask you for this content, show it in their plan and write it once you confirm. You do not need to fill it in by hand.

| Content | Where | Written by |
|---|---|---|
| Features, screens and data models | `docs/project-plan.md` | `/project-plan`, then `/new-feature` |
| Project facts | `AGENTS.md` | `/new-project`, then `/new-feature` |
| Approved deviations from the base | `AGENTS.md` | `/new-feature` |
| The sections at the top of this file | `README.md` | `/new-project`, then `/new-feature` |

### Update the base

`docs/agents/` and `.claude/skills/` are the same in every project, so a project can replace them with a newer version from the base. `AGENTS.md` and this file also hold project-specific content, so update those by hand. Improvements that apply to every project belong in the base.
