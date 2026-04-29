# Action Item: Add EF 6.3 as an Explicit Named Example in Phase 2

## Problem

The constitution agent's Phase 2 already includes a "Prime Example" for OWIN + ASP.NET Identity — a good concrete case that helps the agent recognize over-replacement. However, EF 6.3 is an equally prominent and common case that is not named.

EF 6.3 ships `netstandard2.1` and loads natively on .NET 10 with no compatibility shim needed. Despite this, AI upgrade tools consistently misclassify it as "not compatible" and replace it with Entity Framework Core. EF Core uses a different query syntax, a different migration model, and a different schema convention — the replacement is not a drop-in and often causes data loss or broken queries.

## Context

- **Constitution agent file:**  
  `https://raw.githubusercontent.com/KSchlobohm/fx2dotnet/refs/heads/kschlobohm/setup-agent/agents/02-agent-upgrade-constitution.agent.md`

- **Phase 2 decision standard (current):**
  - Native → PROTECTED
  - Compat-loadable → PROTECTED+NU1701
  - System.Web runtime dep → CORRECTLY GATED
  - Uncertain → STOP AND ASK

- **Current gap:** The only named example is OWIN + Identity (compat-loadable, PROTECTED+NU1701). EF 6.3 (native, PROTECTED) has no named example.

## Requested Change

In Phase 2, alongside or after the OWIN + Identity "Prime Example" block, add:

```
## Prime Example 2: Entity Framework 6.3

EntityFramework 6.3+ ships netstandard2.1. On .NET 10, it loads natively — no compatibility
shim is required and no NU1701 warning will appear.

Common AI misclassification: "EntityFramework does not support .NET 10 — upgrade to
Entity Framework Core."

Why this is wrong: EF 6.x and EF Core are not API-compatible. EF Core uses a different
query model (no ObjectContext, different LINQ translation), a different migration system,
and different schema conventions (e.g., plural vs singular table names, key naming).
Replacing EF 6.3 with EF Core is a data migration project, not a package upgrade.

Correct classification: PROTECTED
Reasoning: Native netstandard2.1 support confirmed. Replacement requires query rewrites,
migration system changes, and potential schema migration — out of scope for this upgrade.
```

## Why This Matters

The retrospective from `c:\dev\assurance-fail` identified EF 6.3 → EF Core replacement as the primary driver of upgrade failure. The agent performed the replacement because nothing in its instructions told it that EF 6.3 was still viable. A named example in Phase 2 gives the agent a pattern to match against and a correct disposition to apply — without needing the user to catch it in the ratification gate.
