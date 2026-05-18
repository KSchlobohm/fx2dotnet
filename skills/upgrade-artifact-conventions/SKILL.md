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
| `{pluginRepoRoot}/feedback/` | Plugin improvement feedback (fx2dotnet repo) |
| `{guideRepoRoot}/feedback/` | Guide improvement feedback (guide repo) |

`{solutionDir}` is the parent directory of the `.sln` file being migrated.

## Artifact Name Patterns

| Artifact | Pattern | Example |
|----------|---------|---------|
| Agent file | `.github/agents/{id}-agent-{purpose}.agent.md` | `.github/agents/01-agent-assessment.agent.md` |
| Sub-agent file | `.github/agents/subagent-{purpose}.agent.md` | `.github/agents/subagent-build-fix.agent.md` |
| Integration test agent | `.github/agents/subagent-integration-test.agent.md` | (fixed name — shared exit gate) |
| Plan file | `{stateRoot}/{id}-plan.md` | `.fx2dotnet/02-plan.md` |
| Progress file (markdown) | `{stateRoot}/{id}-progress.md` | `.fx2dotnet/02-progress.md` |
| Progress file (JSON, script-parsed) | `{stateRoot}/{id}-progress.json` | `.fx2dotnet/05-progress.json` |
| Progress template | `templates/{id}-progress-template.md` | `templates/05-progress-template.md` |
| Phase retrospective | `{stateRoot}/{id}-retrospective.md` | `.fx2dotnet/07-retrospective.md` |
| Phase retrospective addendum | `{stateRoot}/{id}-retrospective-addendum{N}.md` | `.fx2dotnet/04-retrospective-addendum1.md` |
| Phase pre-run notes | `{stateRoot}/{id}-retrospective-prerun.md` | `.fx2dotnet/06-retrospective-prerun.md` |
| Session retrospective | `{stateRoot}/session-retrospective.md` | `.fx2dotnet/session-retrospective.md` |
| Plugin improvement feedback | `{pluginRepoRoot}/feedback/{id}-retrospective.md` | `feedback/06-retrospective.md` |
| Guide improvement feedback | `{guideRepoRoot}/feedback/{id}-retrospective.md` | `feedback/05-retrospective.md` |
| Assessment report | `{stateRoot}/analysis.md` | `.fx2dotnet/analysis.md` |
| Package findings | `{stateRoot}/package-updates.md` | `.fx2dotnet/package-updates.md` |
| Orchestrator state | `{stateRoot}/dotnet-upgrade-plan.md` | `.fx2dotnet/dotnet-upgrade-plan.md` |
| Completion report | `{stateRoot}/08-completion-report.md` | `.fx2dotnet/08-completion-report.md` |
| Constitution | `.github/fx2dotnet/UPGRADE-CONSTITUTION.md` | (fixed path) |

> **Progress file format:** Use `.md` for phases 01–04, whose progress is tracked in prose and not consumed by automation. Use `.json` for phases 05 and above, where a PowerShell validation script reads the file to determine migration status. The schema for each JSON progress file is defined in a corresponding progress template in the `templates/` folder.

## Agent Invocation

The Copilot CLI `--agent` flag strips the `.agent.md` suffix and resolves from `.github/agents/`:

```
--agent 01-agent-assessment
→ .github/agents/01-agent-assessment.agent.md
```

## Rules

1. **Always use the short-ID prefix** — Every phase agent, plan, progress, and retrospective file must be prefixed with its phase ID. Subagents (named `subagent-{purpose}.agent.md`) are shared across phases and are exempt from the phase ID prefix.
2. **State files go in `{stateRoot}`** — No state files at the workspace root or in `.github/`.
3. **Agent files go in `.github/agents/`** — No agent files outside this folder.
4. **Do not use freeform names** — Names like `assessment-agent.md` or `progress.txt` are non-compliant. Use the patterns above.
5. **Do not retroactively rename** — Apply conventions to new artifacts only, unless the user explicitly requests a rename.
6. **Retrospective scope** — Use `{id}-retrospective.md` for a single completed phase. Use `session-retrospective.md` for a cross-phase summary of the full migration run (written after all phases complete or when the migration is suspended). Use `{id}-retrospective-addendum{N}.md` for focused supplements when a single topic warrants deeper treatment than fits in the main retrospective file. Use `{id}-retrospective-prerun.md` for pre-execution checklists or prospective notes written *before* a phase runs.

## What NOT to Do

- Do not save plan files as bare `plan.md` at the workspace root
- Do not name agents without a phase ID prefix (e.g., `assessment.agent.md`)
- Do not place state files in `.github/` or the workspace root
- Do not invent new folder locations — use only the canonical folders above
