# Migration Planner Agent — Recommended Changes

This document captures two gaps found in `migration-planner.agent.md` from the
[KSchlobohm/fx2dotnet](https://github.com/KSchlobohm/fx2dotnet) repository. It is written
as a standalone brief so it can be taken to a separate AI tool or workspace to implement
the changes directly in that repo.

**File to modify:**
```
https://raw.githubusercontent.com/KSchlobohm/fx2dotnet/refs/heads/main/agents/migration-planner.agent.md
```

---

## Change 1: Add "Keep via Compatibility Bridge" as a Resolution Option

### Where in the file

`Step 4 — Resolve Unsupported and Out-of-Scope Packages`

This step currently instructs the planner to recommend exactly one resolution per package
from this list:
- **Replace** — a compatible alternative package exists
- **Remove & rewrite** — functionality can be reimplemented inline
- **Wrap & isolate** — deeply integrated; isolate behind an interface and gate with `#if`
- **Drop** — functionality is no longer needed

### What is missing

**"Keep via compatibility bridge"** is not an option. When the planner encounters
`Microsoft.AspNet.Identity.*` or OWIN packages that have no directly compatible NuGet
version, it currently reaches for "Wrap & isolate" — which gates the Identity types behind
`#if NET48` conditional compilation.

This is the wrong resolution. The `Microsoft.AspNetCore.SystemWebAdapters.Owin` package
provides a bridge that hosts the existing OWIN authentication pipeline inside ASP.NET Core,
preserving `ApplicationUser`, `ApplicationRole`, and the full Identity stack without any
source gating or `#if` changes.

### What went wrong without this option

In a prior migration, "Wrap & isolate" was applied to `ApplicationUser` and `ApplicationRole`
in the Model layer. Those types are used by base classes (`WFEmailBase`, `WFTaskBase`) that
every workflow implementation class inherits from. Gating a base class member propagates
compile errors to all derived classes simultaneously. The result was 850 build errors across
368 files when the Domain layer was compiled against `net10.0` — none of those files
referenced Identity types directly.

If the planner had known about the compatibility bridge option, no gating would have been
written and no cascade would have occurred.

### Recommended change

Add a fifth resolution option to Step 4:

> - **Keep via compatibility bridge** — a shim or adapter package exists that hosts the
>   existing dependency inside the new runtime without source changes. Use when the
>   `owin-identity` skill or another bridge skill applies. Name the bridge package and
>   confirm it is installed in the chunked package update plan.

Also add a note beneath the resolution list:

> **Do not use "Wrap & isolate" for packages where a compatibility bridge exists.** Gating
> a type that is used by shared base classes will propagate compile errors to every derived
> class — a cascade that is invisible until the affected layer is compiled against the new
> target. Check for a bridge option before scheduling any `#if NET48` gating.

---

## Change 2: Read the `owin-identity` Skill Before Resolving Unsupported Packages

### Where in the file

The section that lists inputs the planner reads before beginning its work (the evidence
collection step, equivalent to what the constitution agent calls "Phase 1: Collect Evidence").

### What is missing

The planner does not read `.github/skills/owin-identity/SKILL.md`. The constitution agent
already reads this skill conditionally:

```
### Skills (Domain Policies and Build)
- `.github/skills/owin-identity/SKILL.md` — OWIN bridge policy (if it exists)
```

The planner has no equivalent. This means even when the constitution has established
"keep OWIN/Identity via bridge," the planner doesn't know what bridge to use or how to
plan for it. The constitution's governance principle (retain by default) prevents the
planner from replacing Identity, but without the skill reference it has no lighter-weight
path to choose instead of gating.

### Recommended change

Add the following to the planner's input/evidence reading section, using the same
conditional pattern as the constitution agent:

> - `.github/skills/owin-identity/SKILL.md` — OWIN bridge policy (read if it exists;
>   apply its guidance when resolving any package under `Microsoft.AspNet.Identity.*`,
>   `Microsoft.Owin.*`, or related OWIN namespaces)

And add a note in Step 4 near the unsupported package resolution instructions:

> **Before resolving any OWIN or ASP.NET Identity package:** read
> `.github/skills/owin-identity/SKILL.md` if it exists. If the skill is present, its
> guidance takes precedence over the default resolution options for those packages. The
> correct approach is the `Microsoft.AspNetCore.SystemWebAdapters.Owin` compatibility
> bridge — not Replace, not Wrap & isolate.

---

## Summary of Changes

| # | Location in file | Change |
|---|-----------------|--------|
| 1 | Step 4 resolution options | Add "Keep via compatibility bridge" as a fifth option |
| 1 | Step 4 notes | Add warning against using Wrap & isolate when a bridge exists |
| 2 | Input/evidence reading section | Add conditional read of `.github/skills/owin-identity/SKILL.md` |
| 2 | Step 4 pre-resolution note | Instruct planner to apply owin-identity skill for OWIN/Identity packages |

Both changes are additive — no existing instructions need to be removed.
