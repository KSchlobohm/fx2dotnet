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

Declare **only the tools the plan actually needs**. Review the plan before populating the `tools` list — do not use a generic catch-all list.

## Agent Structure

Every phase agent must include these sections:

### 1. Resume Check

Before starting any work, check whether a progress file exists at `{stateRoot}/{id}-progress.md`:
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

The agent must read its plan file at the start of execution:

```
Read {stateRoot}/{id}-plan.md before beginning any work.
```

The agent executes the plan as written. It does not reinterpret or skip steps.

## What NOT to Do

- Do not hardcode file paths in the agent — derive them from `{stateRoot}` and `{id}`
- Do not declare tools not needed by the plan
- Do not skip the resume check — agents must be safe to restart
- Do not proceed past a failed phase without stopping and reporting the failure
- Do not mark a phase complete without test evidence
