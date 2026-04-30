---
name: upgrade-artifact-conventions
description: "Canonical naming and placement conventions for all fx2dotnet migration artifacts. Use when: creating agents, plan files, progress files, retrospective files, or any state artifact during a .NET Framework to modern .NET migration. Defines the short-ID format, folder locations, and predictable file name patterns that all phases must follow."
---

# Artifact Naming and Placement Conventions

## Purpose

These conventions ensure every agent, plan, progress file, and retrospective is named and placed predictably. Any agent or human can locate any artifact without searching.

These are **forward-looking** conventions. They govern artifacts created from this point forward and do not require renaming files that already exist.

## Short-ID Format

Every migration phase uses a short, human-typeable ID:

```
{phase}{optional-letter}
```

| Example ID | Meaning |
|------------|---------|
| `00`       | Workspace setup |
| `01`       | Assessment |
| `02`       | Upgrade constitution |
| `02b`      | Constitution sub-phase b |
| `05c`      | Phase 5, sub-phase c |

Use the lowest-specificity ID that is unambiguous. Add a letter suffix only when a phase has parallel or sequential sub-steps.

## Canonical Folders

| Folder | Contents |
|--------|----------|
| `.github/agents/` | All agent files for this workspace |
| `.github/skills/` | All skill files for this workspace |
| `.github/fx2dotnet/` | Governance documents (constitution, amendments) |
| `{solutionDir}/.fx2dotnet/` | All state files (`{stateRoot}`) |

`{solutionDir}` is the parent directory of the `.sln` file being migrated.

## Artifact Name Patterns

| Artifact | Pattern | Example |
|----------|---------|---------|
| Agent file | `.github/agents/{id}-agent-{purpose}.agent.md` | `.github/agents/01-agent-assessment.agent.md` |
| Sub-agent file | `.github/agents/subagent-{purpose}.agent.md` | `.github/agents/subagent-build-fix.agent.md` |
| Plan file | `{stateRoot}/{id}-plan.md` | `.fx2dotnet/02-plan.md` |
| Progress file | `{stateRoot}/{id}-progress.md` | `.fx2dotnet/02-progress.md` |
| Retrospective | `{stateRoot}/{id}-retrospective.md` | `.fx2dotnet/02-retrospective.md` |
| Assessment report | `{stateRoot}/analysis.md` | `.fx2dotnet/analysis.md` |
| Package findings | `{stateRoot}/package-updates.md` | `.fx2dotnet/package-updates.md` |
| Constitution | `.github/fx2dotnet/UPGRADE-CONSTITUTION.md` | (fixed path) |

## Agent Invocation

The Copilot CLI `--agent` flag strips the `.agent.md` suffix and resolves from `.github/agents/`:

```
--agent 01-agent-assessment
→ .github/agents/01-agent-assessment.agent.md
```

## Rules

1. **Always use the short-ID prefix** — Every agent, plan, progress, and retrospective file must be prefixed with its phase ID.
2. **State files go in `{stateRoot}`** — No state files at the workspace root or in `.github/`.
3. **Agent files go in `.github/agents/`** — No agent files outside this folder.
4. **Do not use freeform names** — Names like `plan.md`, `assessment-agent.md`, or `progress.txt` are non-compliant. Use the patterns above.
5. **Do not retroactively rename** — Apply conventions to new artifacts only, unless the user explicitly requests a rename.

## What NOT to Do

- Do not save plan files as bare `plan.md` at the workspace root
- Do not name agents without a phase ID prefix (e.g., `assessment.agent.md`)
- Do not place state files in `.github/` or the workspace root
- Do not invent new folder locations — use only the canonical folders above
