# Package Update Progress Template

This template defines the structure of `.fx2dotnet/05-progress.json` — the progress tracking
file for Phase 2 (Package Compatibility). The package update agent reads and writes this file
to track which chunks are complete, in progress, or deferred.

## How It Works

1. **Step 1 (agent creation)** produces the initial progress file from the plan — all chunks start as `pending`.
2. **Each agent invocation** reads the file, finds the next `pending` chunk, processes it, and updates the status.
3. **The validation script** parses this file to report what's done vs. what remains.

## Schema

```json
{
  "phase": "05-package-updates",
  "sourcePlan": ".fx2dotnet/05-plan.md",
  "testTargetProject": "relative/path/to/LegacyHost.csproj",
  "lastUpdated": "2026-05-04T14:30:00Z",
  "updatedBy": "agent | manual",
  "chunks": [
    {
      "id": 1,
      "description": "short description from plan",
      "packages": [
        { "name": "PackageName", "from": "1.0.0", "to": "2.0.0" }
      ],
      "status": "pending",
      "validation": null,
      "notes": null
    }
  ]
}
```

## Field Definitions

### Root

| Field | Type | Description |
|-------|------|-------------|
| `phase` | string | Always `"05-package-updates"` |
| `sourcePlan` | string | Relative path to the plan file |
| `testTargetProject` | string | Relative path to the `.csproj` used for integration test validation. For phases 05–06 this is the legacy host; the Integration Test agent uses this as the authoritative project path. |
| `lastUpdated` | string (ISO 8601) | Timestamp of last modification |
| `updatedBy` | string | `"agent"` or `"manual"` |
| `chunks` | array | Ordered list of chunk objects |

### Chunk Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Chunk number from plan (1-based, sequential) |
| `description` | string | Brief label for the chunk's scope |
| `packages` | array | Package changes in this chunk (name, from version, to version) |
| `status` | string | `"pending"` · `"in-progress"` · `"done"` · `"blocked"` · `"deferred"` |
| `validation` | string or null | `"pass"` · `"fail"` · `null` (not yet run) |
| `notes` | string or null | Failure reason, deferral rationale, or commit SHA |

## Status Definitions

| Status | Meaning |
|--------|---------|
| `pending` | Chunk has not been attempted |
| `in-progress` | Agent is currently working on this chunk |
| `done` | All packages in this chunk updated and validation passed |
| `blocked` | Chunk attempted but cannot be completed; requires human intervention |
| `deferred` | Chunk intentionally skipped per plan guidance |

## Rules

1. **One chunk at a time.** The agent sets the next `pending` chunk to `in-progress`, processes it, then sets it to `done`, `blocked`, or `deferred`.
2. **No skipping.** Chunks are processed in order unless the plan explicitly allows parallel chunks.
3. **Blocked stops the loop.** If a chunk is `blocked`, the agent must stop. The human reviews and either resolves the block or changes the status to `deferred` before rerunning.
4. **Deferred is terminal.** A `deferred` chunk is not retried in this phase.
5. **Validation is mandatory.** The agent must record `validation: "pass"` before setting `status: "done"`.

## Example: Completed Progress File

```json
{
  "phase": "05-package-updates",
  "sourcePlan": ".fx2dotnet/05-plan.md",
  "lastUpdated": "2026-05-04T16:45:00Z",
  "updatedBy": "agent",
  "chunks": [
    {
      "id": 1,
      "description": "EF6 ecosystem (EntityFramework 6.3.0, DynamicFilters 3.1.0)",
      "packages": [
        { "name": "EntityFramework", "from": "6.2.0", "to": "6.3.0" },
        { "name": "EntityFramework.DynamicFilters", "from": "3.0.1", "to": "3.1.0" }
      ],
      "status": "done",
      "validation": "pass",
      "notes": "commit abc1234"
    },
    {
      "id": 2,
      "description": "Independent minor upgrades",
      "packages": [
        { "name": "log4net", "from": "2.0.8", "to": "2.0.10" },
        { "name": "Ether.Outcomes", "from": "2.2.0", "to": "2.9.5" },
        { "name": "EPPlus", "from": "4.1.1", "to": "4.5.1" },
        { "name": "Microsoft.AspNet.WebApi.Client", "from": "5.2.3", "to": "5.2.4" },
        { "name": "Microsoft.Data.Services.Client", "from": "5.6.4", "to": "5.8.2" }
      ],
      "status": "done",
      "validation": "pass",
      "notes": "commit def5678"
    },
    {
      "id": 3,
      "description": "Z.EntityFramework.Extensions major version jump",
      "packages": [
        { "name": "Z.EntityFramework.Extensions", "from": "4.0.93", "to": "7.100.0" }
      ],
      "status": "deferred",
      "validation": null,
      "notes": "Breaking API changes exceed session scope; deferred to post-migration"
    }
  ]
}
```
