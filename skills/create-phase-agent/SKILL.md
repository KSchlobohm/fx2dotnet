---
name: create-phase-agent
description: "Pattern for creating a phase execution agent from a plan file. Use when: asked to create an agent to execute a plan, implement a phase, or carry out assessment/migration work. Defines required frontmatter, file naming, plan execution behavior, progress tracking, test evidence, and phase structure that all phase agents must follow."
---

# Create Phase Agent

## Purpose

This skill governs how phase execution agents are created. Apply it whenever asked to create an agent that will carry out a plan or execute a migration phase.

## Naming and Placement

Follow the `upgrade-artifact-conventions` skill for all naming decisions:

- **File location:** `.github/agents/`
- **File name:** `{id}-agent-{purpose}.agent.md`
- **Frontmatter `name` property:** `"{id} {Purpose}"` — phase ID prefix followed by a short human-readable label

Examples:
- File: `.github/agents/01-agent-assessment.agent.md` → `name: "01 Assessment"`
- File: `.github/agents/04-agent-sdk-project-conversion.agent.md` → `name: "04 SDK Project Conversion"`

## Required Frontmatter

```yaml
---
name: "{Id} Agent {Purpose}"
description: "One sentence describing what this agent does and when to use it."
tools: [<tools required by the plan>]
---
```

**Template-first rule**: When creating a workspace-specific agent that corresponds to a plugin template agent (e.g., `agents/01-agent-assessment.agent.md`), **start from the template's `tools:` list** and remove only what the specific plan genuinely doesn't need. Never build the `tools:` list from scratch by reading only the plan — MCP namespace declarations are easy to miss in plan text and are already correctly specified in the template.

**MCP tool mapping rule**: For every MCP tool call referenced in the plan, its server namespace must appear in `tools:` using the `namespace/*` wildcard. The Copilot agent runtime only connects MCP servers that are explicitly declared — omitting a namespace means those tools are never callable, and the agent will silently fall back to manual work without any error.

Example: a plan that calls `generate_dotnet_upgrade_assessment` and `FindRecommendedPackageUpgrades` requires:

```yaml
tools: [microsoft.githubcopilot.appmodernization.mcp/*, Swick.Mcp.Fx2dotnet/*, ...]
```

**`agent` tool rule**: Include `agent` in `tools:` whenever the plan delegates any work to sub-agents — not only for build/fix loops. Without `agent`, the runtime cannot spawn sub-agents and the orchestrator will be forced to do all work inline.

## Agent Structure

Every phase agent must include these sections:

### 1. Resume Check

Before starting any work, locate the relevant state and resume from the last incomplete step.

**For phases that process multiple projects** (e.g., SDK conversion, multitarget, web migration): state is tracked in per-project sections inside `{stateRoot}/{ProjectName}.md`. Check whether the relevant section (e.g., `## SDK Conversion`) already exists and is marked complete before processing each project.

**For phases that produce a single artifact** (e.g., assessment, constitution): check whether a progress file exists at `{stateRoot}/{id}-progress.md`:
- If it exists, read it and resume from the last incomplete phase
- If it does not exist, create it and start from Phase 1

### 2. Numbered Phases

Break all work into explicit numbered phases that map directly to the plan:

```
## Phase 1: {Name}
## Phase 2: {Name}
...
```

Each phase must:
- Have a single, clearly bounded unit of work
- Write its completion status to the progress file before moving to the next phase
- Be resumable independently if the agent is interrupted

### 3. Progress Tracking

After completing each phase, update `{stateRoot}/{id}-progress.md`:

```markdown
## Phase 1: {Name} — complete
## Phase 2: {Name} — complete
## Phase 3: {Name} — in progress
```

Never mark a phase complete before verifying its outputs exist and are valid.

### 4. Test Evidence

At every phase that produces or modifies an artifact, provide explicit evidence:
- Quote or summarize the key output (file path, record count, classification result, etc.)
- If a build or test was run, include the pass/fail result
- Do not proceed to the next phase if evidence is missing or a check fails

### 5. Plan Reference

#### Plan Discovery

Before reading a plan file, the agent must discover which plan applies using this priority order:

1. **Examine pending git commits**: Run `git log --oneline -10` and `git status --short` in the workspace. Look for:
   - Commit messages that reference a plan file, phase name, or `.fx2dotnet/` state
   - Staged or modified files under `.fx2dotnet/` that indicate an in-progress phase

   If git reveals an explicit plan file path or a clear phase context, use that file as the plan.

2. **Fall back to convention**: If git does not yield a clear signal, look for `{stateRoot}/{id}-plan.md`, where `{id}` is the numeric prefix of this agent's file name (e.g., `01` for `01-agent-assessment.agent.md` → `{stateRoot}/01-plan.md`).

Once the plan file is identified, read it before beginning any work. The agent executes the plan as written. It does not reinterpret or skip steps.

## Subagents

Phase agents that involve building, compiling, or verifying build health must declare the `Build Fix` agent in their frontmatter and delegate all build/fix loops to it:

```yaml
agents: ['Build Fix']
```

Do not implement a build/fix loop inline inside a phase agent — delegate to `Build Fix` instead.

## Sub-Agent Delegation for Research-Heavy Phases

When a phase requires reading many files or building large inventory tables (e.g., enumerating all `.csproj` files in a solution, triaging all packages across every `packages.config`), delegate that work to an `explore` or `general-purpose` sub-agent rather than reading files inline in the orchestrator context.

Accumulating raw file content across many files in a single context window risks triggering a compaction event, which truncates conversation history and degrades the agent's ability to reason over its own prior findings.

**Pattern**: the orchestrator launches research sub-agents that return a **summary table or structured result** — the orchestrator writes that result to disk. The orchestrator should synthesize and persist findings; it should not accumulate raw file content.

Phases that are mutually independent (their findings do not depend on each other) can be launched as concurrent sub-agents to reduce total elapsed time.

## What NOT to Do

- Do not hardcode file paths in the agent — derive them from `{stateRoot}` and `{id}`
- Do not declare tools not needed by the plan
- Do not skip the resume check — agents must be safe to restart
- Do not proceed past a failed phase without stopping and reporting the failure
- Do not mark a phase complete without test evidence
