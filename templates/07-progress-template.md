# ASP.NET Web Migration Progress Template

This template defines the structure of `.fx2dotnet/07-progress.json` — the progress tracking
file for Phase 07 (ASP.NET Web Migration). The web migration agent reads and writes this file
to track which slices are complete, in progress, or deferred.

## How It Works

1. **Step 1 (plan phase)** produces the initial progress file from the migration plan — all slices start as `pending`.
2. **Each agent invocation** reads the file, finds the next `pending` slice, ports the artifacts, and updates the status.
3. **The validation script** parses this file to report what's done vs. what remains.

## Schema

```json
{
  "phase": "07-aspnet-web-migration",
  "sourcePlan": ".fx2dotnet/07-plan.md",
  "legacyProject": "relative/path/to/LegacyWeb.csproj",
  "newHostProject": "relative/path/to/NewWeb.csproj",
  "lastUpdated": "2026-05-04T14:30:00Z",
  "updatedBy": "agent | manual",
  "slices": [
    {
      "id": 1,
      "description": "short description from plan",
      "endpoints": [
        { "method": "GET", "route": "/api/example", "controller": "ExampleController", "action": "Get" }
      ],
      "status": "pending",
      "buildValidation": null,
      "notes": null
    }
  ]
}
```

## Field Definitions

### Root

| Field | Type | Description |
|-------|------|-------------|
| `phase` | string | Always `"07-aspnet-web-migration"` |
| `sourcePlan` | string | Relative path to the plan file |
| `legacyProject` | string | Relative path to the legacy ASP.NET `.csproj` file. Retained for side-by-side behavioral comparison — agents must never delete this project. |
| `newHostProject` | string | Relative path to the new ASP.NET Core `.csproj` file. This is the integration test target for phase 07+. |
| `lastUpdated` | string (ISO 8601) | Timestamp of last modification |
| `updatedBy` | string | `"agent"` or `"manual"` |
| `slices` | array | Ordered list of slice objects |

### Slice Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Slice number from plan (1-based, sequential) |
| `description` | string | Brief label for the slice's scope (e.g., "Host bootstrap and DI", "Auth middleware") |
| `endpoints` | array or null | Endpoints ported in this slice; `null` for infrastructure slices with no direct endpoint surface |
| `status` | string | `"pending"` · `"in-progress"` · `"done"` · `"blocked"` · `"deferred"` |
| `buildValidation` | string or null | `"pass"` · `"fail"` · `null` (not yet run) |
| `notes` | string or null | Failure reason, deferral rationale, adapter decisions, or commit SHA |

### Endpoint Object

| Field | Type | Description |
|-------|------|-------------|
| `method` | string | HTTP method (e.g., `"GET"`, `"POST"`) |
| `route` | string | Route template (e.g., `"/api/users/{id}"`) |
| `controller` | string | Source controller class name |
| `action` | string | Source action method name |

## Status Definitions

| Status | Meaning |
|--------|---------|
| `pending` | Slice has not been attempted |
| `in-progress` | Agent is currently working on this slice |
| `done` | Slice artifacts ported and new host builds cleanly |
| `blocked` | Slice attempted but cannot be completed; requires human intervention |
| `deferred` | Slice intentionally skipped per plan guidance |

## Validation Rules

A slice is considered validated (`"pass"`) when:

1. The new ASP.NET Core host project (`newHostProject`) builds cleanly after the slice is applied — validated by delegating to the `Build Fix` agent.
2. All in-scope endpoints from the slice are implemented, intentionally retired, or explicitly deferred.

## Recommended Slice Order

The agent should process slices in this order (from `07-agent-aspnet-web-migration.agent.md`):

1. Host bootstrap, configuration, and dependency injection
2. Cross-cutting middleware and filters
3. Authentication and authorization
4. Serialization, validation, and exception handling
5. Controllers and endpoint mappings (one controller or route group per slice)
6. OpenAPI, health checks, CORS, static files, and operational features

## Rules

1. **One slice at a time.** The agent sets the next `pending` slice to `in-progress`, processes it, then sets it to `done`, `blocked`, or `deferred`.
2. **Build Fix after every slice.** The agent must delegate to `Build Fix` before recording `buildValidation: "pass"` and before proceeding to the next slice.
3. **Blocked stops the loop.** If a slice is `blocked`, the agent must stop. The human reviews and either resolves the block or changes the status to `deferred` before rerunning.
4. **Deferred is terminal.** A `deferred` slice is not retried in this phase.
5. **Endpoint parity is mandatory.** The agent must record `buildValidation: "pass"` before setting `status: "done"`.

## Example: Completed Progress File

```json
{
  "phase": "07-aspnet-web-migration",
  "sourcePlan": ".fx2dotnet/07-plan.md",
  "legacyProject": "src/MyApp.Web/MyApp.Web.csproj",
  "newHostProject": "src/MyApp.WebCore/MyApp.WebCore.csproj",
  "lastUpdated": "2026-05-04T16:45:00Z",
  "updatedBy": "agent",
  "slices": [
    {
      "id": 1,
      "description": "Host bootstrap, configuration, and dependency injection",
      "endpoints": null,
      "status": "done",
      "buildValidation": "pass",
      "notes": "commit abc1234"
    },
    {
      "id": 2,
      "description": "Authentication and authorization middleware",
      "endpoints": null,
      "status": "done",
      "buildValidation": "pass",
      "notes": "Used systemweb-adapters for HttpContext during migration; commit def5678"
    },
    {
      "id": 3,
      "description": "UsersController endpoints",
      "endpoints": [
        { "method": "GET", "route": "/api/users", "controller": "UsersController", "action": "GetAll" },
        { "method": "GET", "route": "/api/users/{id}", "controller": "UsersController", "action": "GetById" },
        { "method": "POST", "route": "/api/users", "controller": "UsersController", "action": "Create" }
      ],
      "status": "done",
      "buildValidation": "pass",
      "notes": "commit ghi9012"
    },
    {
      "id": 4,
      "description": "OrdersController endpoints",
      "endpoints": [
        { "method": "GET", "route": "/api/orders/{id}", "controller": "OrdersController", "action": "GetById" }
      ],
      "status": "blocked",
      "buildValidation": "fail",
      "notes": "Depends on IOrderRepository which is not yet available on net10.0; requires human review"
    }
  ]
}
```
