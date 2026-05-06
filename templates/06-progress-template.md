# Multitarget Migration Progress Template

This template defines the structure of `.fx2dotnet/06-progress.json` — the progress tracking
file for Phase 06 (Multitarget Migration). The multitarget agent reads and writes this file
to track which layers are complete, in progress, or deferred.

## How It Works

1. **Step 1 (agent creation)** produces the initial progress file from the plan — all layers start as `pending`.
2. **Each agent invocation** reads the file, finds the next `pending` layer, processes it, and updates the status.
3. **The validation script** parses this file to report what's done vs. what remains.

## Schema

```json
{
  "phase": "06-multitarget",
  "sourcePlan": ".fx2dotnet/06-plan.md",
  "lastUpdated": "2026-05-04T14:30:00Z",
  "updatedBy": "agent | manual",
  "layers": [
    {
      "id": 1,
      "description": "short description from plan",
      "projects": [
        { "path": "relative/path/to/Project.csproj", "from": "net48", "to": "net48;net10.0" }
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
| `phase` | string | Always `"06-multitarget"` |
| `sourcePlan` | string | Relative path to the plan file |
| `lastUpdated` | string (ISO 8601) | Timestamp of last modification |
| `updatedBy` | string | `"agent"` or `"manual"` |
| `layers` | array | Ordered list of layer objects |

### Layer Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Layer number from plan (1-based, sequential) |
| `description` | string | Brief label for the layer's scope |
| `projects` | array | Projects in this layer (path, from TFM, to TFM) |
| `status` | string | `"pending"` · `"in-progress"` · `"done"` · `"blocked"` · `"deferred"` |
| `validation` | string or null | `"pass"` · `"fail"` · `null` (not yet run) |
| `notes` | string or null | Failure reason, deferral rationale, or commit SHA |

### Project Object

| Field | Type | Description |
|-------|------|-------------|
| `path` | string | Relative path to the `.csproj` file from the solution directory |
| `from` | string | Original target framework (e.g., `"net48"`) |
| `to` | string | New multitarget value (e.g., `"net48;net10.0"`) |

## Status Definitions

| Status | Meaning |
|--------|---------|
| `pending` | Layer has not been attempted |
| `in-progress` | Agent is currently working on this layer |
| `done` | All projects in this layer updated and both build targets validated |
| `blocked` | Layer attempted but cannot be completed; requires human intervention |
| `deferred` | Layer intentionally skipped per plan guidance |

## Validation Rules

A layer is considered validated (`"pass"`) only when both of the following succeed for every project in the layer:

1. `dotnet build -f net48` — the .NET Framework target builds without errors.
2. `dotnet build -f net10.0` — the modern .NET target builds without errors.

## Rules

1. **One layer at a time.** The agent sets the next `pending` layer to `in-progress`, processes it, then sets it to `done`, `blocked`, or `deferred`.
2. **No skipping.** Layers are processed in order unless the plan explicitly allows parallel layers.
3. **Blocked stops the loop.** If a layer is `blocked`, the agent must stop. The human reviews and either resolves the block or changes the status to `deferred` before rerunning.
4. **Deferred is terminal.** A `deferred` layer is not retried in this phase.
5. **Both build targets required.** The agent must record `validation: "pass"` (both `net48` and `net10.0` succeed) before setting `status: "done"`.

## Example: Completed Progress File

```json
{
  "phase": "06-multitarget",
  "sourcePlan": ".fx2dotnet/06-plan.md",
  "lastUpdated": "2026-05-04T16:45:00Z",
  "updatedBy": "agent",
  "layers": [
    {
      "id": 1,
      "description": "Core domain library",
      "projects": [
        { "path": "src/MyApp.Domain/MyApp.Domain.csproj", "from": "net48", "to": "net48;net10.0" }
      ],
      "status": "done",
      "validation": "pass",
      "notes": "commit abc1234"
    },
    {
      "id": 2,
      "description": "Infrastructure and data access",
      "projects": [
        { "path": "src/MyApp.Data/MyApp.Data.csproj", "from": "net48", "to": "net48;net10.0" },
        { "path": "src/MyApp.Infrastructure/MyApp.Infrastructure.csproj", "from": "net48", "to": "net48;net10.0" }
      ],
      "status": "done",
      "validation": "pass",
      "notes": "commit def5678"
    },
    {
      "id": 3,
      "description": "Shared utilities with platform-specific dependencies",
      "projects": [
        { "path": "src/MyApp.Utilities/MyApp.Utilities.csproj", "from": "net48", "to": "net48;net10.0" }
      ],
      "status": "blocked",
      "validation": "fail",
      "notes": "Uses System.Drawing which requires conditional compilation guards; requires human review"
    }
  ]
}
```
