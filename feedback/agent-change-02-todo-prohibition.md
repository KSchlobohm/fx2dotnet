# Action Item: Add `// TODO` Prohibition as a Constitution Principle

## Problem

The constitution agent's "Forbidden Actions" section governs what execution agents must not do with protected packages at runtime. But it does not address how deferred work must be tracked.

A retrospective finding from `c:\dev\assurance-fail` showed that developers (and AI agents) routinely use `// TODO` source comments or `#if NET48` preprocessor gates to defer responsibility. These are invisible to any agent that executes a future phase — agents read plan files, not source comments. This means deferred work silently disappears from the upgrade plan.

## Context

- **Constitution agent file:**  
  `https://raw.githubusercontent.com/KSchlobohm/fx2dotnet/refs/heads/kschlobohm/setup-agent/agents/02-agent-upgrade-constitution.agent.md`

- **Retrospective source:** `c:\dev\assurance-fail` — the soft-delete regression case, where `EntityFramework.DynamicFilters` was gated on `#if NET48` without a corresponding plan entry, silently dropping the soft-delete filter with no compile error and no warning.

- **Where this principle belongs:** Phase 3 (Draft) of the constitution agent, in the section that governs execution standards — alongside or inside the "Forbidden Actions" list.

## Requested Change

Add the following as an explicit rule in the constitution agent's Phase 3 "Forbidden Actions" (or a new "Deferred Work Standards" section):

> **`// TODO` and `#if` gates are not deferred-work records.**  
> Any capability that is gated behind `#if NET48`, commented out, or marked `// TODO` during the upgrade must have a corresponding named entry in a plan file before the commit is made.  
> The plan file must name: (1) the capability being deferred, (2) the package or code path involved, (3) the condition under which it will be addressed.  
> Source comments are not visible to agents executing future phases. If it is not in a plan file, it does not exist.

This rule should also be written into the output `CONSTITUTION.md` file so all execution agents that read it inherit the constraint.

## Why This Matters

`// TODO` stubs give the appearance of safety but provide none. An agent executing Phase 5 (SDK Conversion) will not scan source files for `// TODO` comments before deciding what to change. If a soft-delete filter, an authorization middleware, or a logging hook was deferred via a comment, that agent has no way to know it exists. The only reliable handoff mechanism between phases is named plan files.
