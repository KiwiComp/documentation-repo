# AGENTS.md

## Read first

These documents hold the rules, and this file does not repeat them. Read the ones that apply before you start.

| Document | Read |
|---|---|
| `docs/agents/working-conventions.md` | For every task |
| `docs/agents/architecture-principles.md` | Before writing or reviewing code |
| `docs/agents/testing.md` | Before writing or reviewing code or tests |
| `docs/project-plan.md`, if it exists | Before planning a feature |

## Skills

The instructions are in `.claude/skills/<name>/SKILL.md`. If your tool does not support skills, read that file and follow it when the skill is requested.

| Skill | Use | Started by |
|---|---|---|
| `/project-plan` | Plan the app's features, screens and data models | Developer only |
| `/new-project` | Set up the app-wide foundation in a new project | Developer only |
| `/new-feature` | Clarify, plan and implement a new feature | Developer only |
| `/verify` | Check the project after every code change | Agent or developer |
| `/pull-request` | Commit, push and open a pull request | Developer only |

## Project facts

`/new-project` fills in this section. If it is still `<not set>`, suggest `/new-project`. Where this file and `docs/agents/` differ, this file wins. Keep it to facts that cannot be read from the code, and link to larger project docs under `docs/` instead of adding them here.

- App: <not set>
- Supported languages: <not set>
- Tabs: <not set>
- Room: <not set>
- Network: <not set>

## Approved deviations

Lasting deviations from `docs/agents/` that the developer has approved for this project, each with its reason.

- None
