# Action Item: Make Protected Dependency Matrix Planner-Consumable

## Problem

The constitution agent (`02-agent-upgrade-constitution.agent.md`) produces a Protected Dependency Matrix in Phase 3, but it is formatted as informational prose. There is no instruction to the agent to format the matrix as a structured, machine-readable constraint block that a downstream planner agent can treat as a hard exclusion list.

Without this, the planner may read `UPGRADE-CONSTITUTION.md` and still schedule work items against protected packages — because nothing tells it those entries are binding constraints, not just observations.

## Context

- **Constitution agent file:**  
  `https://raw.githubusercontent.com/KSchlobohm/fx2dotnet/refs/heads/kschlobohm/setup-agent/agents/02-agent-upgrade-constitution.agent.md`

- **Where the matrix is produced:** Phase 3 (Draft) of the constitution agent — the agent classifies each dependency and assigns a disposition (PROTECTED, PROTECTED+NU1701, CORRECTLY GATED, etc.)

- **Where the matrix needs to be consumed:** The planning agent (Phase 03 of the workshop guide) — it should read `UPGRADE-CONSTITUTION.md` and skip any work item that touches a protected package.

- **Current gap:** The constitution agent does not instruct the output file to include a dedicated planner-facing section. The planner agent does not (yet) have an instruction to read the constitution before creating work items.

## Requested Changes

### Change 1: Constitution agent — Phase 3 output format

In the Phase 3 (Draft) instructions, add a requirement that `UPGRADE-CONSTITUTION.md` must include a dedicated section with a fixed heading, e.g.:

```markdown
## Planner Constraints

The following packages are protected. The planning agent MUST NOT create any work item
that modifies, replaces, or removes these packages.

| Package | Disposition | Reason |
|---------|-------------|--------|
| EntityFramework | PROTECTED | Ships netstandard2.1; native on .NET 10; replacement requires schema migration |
| Microsoft.AspNet.Identity.* | PROTECTED+NU1701 | Compat-loadable; replacement requires incompatible SQL schema changes |
```

The fixed heading `## Planner Constraints` allows the planner to locate it reliably with a grep or section parse.

### Change 2: Planning agent — add a pre-condition check

In the planner agent file (whichever file drives Phase 03), add an early instruction:

> Before creating any work items, read `.github/fx2dotnet/UPGRADE-CONSTITUTION.md`.  
> Locate the `## Planner Constraints` section.  
> Do not create any work item — upgrade, removal, or replacement — for any package listed in that table.  
> If a work item would otherwise target a protected package, drop it silently and note the omission in the plan summary.

## Why This Matters

The retrospective from `c:\dev\assurance-fail` showed that AI upgrade tools replaced EF 6.3 with EF Core and OWIN/ASP.NET Identity with ASP.NET Core Identity — both decisions that caused incompatible SQL schema changes and broken authentication. These packages were not "unmanageable" — they load natively or via compat on .NET 10. They were replaced because no constraint existed to prevent it. The constitution phase was designed to create that constraint. This change makes the constraint binding, not advisory.
