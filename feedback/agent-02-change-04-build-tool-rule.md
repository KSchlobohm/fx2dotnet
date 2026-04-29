# Action Item: Strengthen the Build Tool Rule in Phase 1

## Problem

The constitution agent's Phase 1 (Evidence) instructs the agent to run a full solution build as part of the build census. However, it does not specify which build tool to use. For solutions that still target `.NET Framework`, using `dotnet build` with a framework moniker (`-f net48`) can succeed while masking errors that only surface during a full `msbuild` build of the `.sln` file.

This means the "baseline build passes" evidence collected in Phase 1 may be unreliable — the constitution could be drafted on the assumption that the solution builds cleanly when it does not.

## Context

- **Constitution agent file:**  
  `https://raw.githubusercontent.com/KSchlobohm/fx2dotnet/refs/heads/kschlobohm/setup-agent/agents/02-agent-upgrade-constitution.agent.md`

- **Phase 1 goal:** Collect build census evidence — which projects build, which fail, which have multi-targeting.

- **The gap:** `dotnet build -f net48` evaluates individual projects in isolation and may not surface inter-project dependencies, missing SDK resolution, or project reference issues that `msbuild <solution>.sln` would catch.

- **Retrospective source:** `c:\dev\assurance-fail` — build evidence was collected with `dotnet build` per-project; full-solution build issues were not surfaced until execution.

## Requested Change

In Phase 1 (Evidence), update the build census instructions to specify:

> **Build tool requirement:**  
> Use `msbuild <solution>.sln /t:Build /p:Configuration=Debug` for full-solution validation.  
> Do not use `dotnet build -f net48` as a substitute — it evaluates projects in isolation and may not surface inter-project or SDK resolution errors.  
> If `msbuild` is not available in the current environment, stop and report this as a blocking prerequisite.

Additionally, add this as a principle in the output `UPGRADE-CONSTITUTION.md` so execution agents inherit it:

> All build validation during this upgrade must use `msbuild <solution>.sln` for full-solution builds.  
> Per-project `dotnet build` is permitted only for targeted diagnostic checks, not as evidence of solution-wide build health.

## Why This Matters

The constitution phase is the last point where baseline evidence is collected before execution begins. If the build baseline is recorded using an unreliable tool, errors that surface during execution will be misattributed to the upgrade rather than to pre-existing issues. Specifying `msbuild` as the required tool ensures the baseline is accurate and execution agents can trust that a failing build during upgrade is a regression, not a pre-existing condition.
