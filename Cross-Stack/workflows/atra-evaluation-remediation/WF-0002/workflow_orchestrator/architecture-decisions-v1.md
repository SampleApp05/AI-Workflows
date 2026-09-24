---
artifact:
  id: DEC-0001
  type: DECISION_RECORD
  version: 1
  status: APPROVED
workflow:
  id: WF-0002
  project: Atra
  technology: Cross-Stack
  feature: atra-evaluation-remediation
  mode: delivery
  contract_version: "1.2"
lineage:
  parents:
    - ARC-0002@v1
ownership:
  created_by: workflow_orchestrator
  performed_by: human
  recorded_by: workflow_orchestrator
  engine: codex
  target_id: codex-native
---

# Architecture Decisions — Atra evaluation remediation

## Source

The requesting human provided these decisions in this task at `2026-09-24T07:43:12Z`.

## Decisions

1. **Market freshness interface:** no existing consumers exist. The `/prices` contract may change directly to expose the approved freshness semantics; no API versioning, parallel endpoint, or compatibility path is required.
2. **Incompatible existing role/owner data:** use the recommended safety approach. If an authorized preflight finds duplicate roles, multiple owners or recovery roles, or a canonical-owner mismatch, stop and report evidence. This run is not authorized to repair production data or otherwise treat incompatible existing data.
3. **Session lifecycle:** use the recommended policy. Refresh preserves the original authenticating wallet; a user may revoke their own session; recovery, ownership transfer, or loss of an authorizing role revokes affected sessions.
4. **Verification environment:** a disposable, non-production PostgreSQL environment is authorized for applying workflow migrations and running competing-write tests. This authorizes no production connection, data operation, deployment, or persistent environment change.

## Effect

These decisions resolve the material Architecture-stage choices documented in `ARC-0002@v1`. They permit a bounded same-stage Architecture revision and later requirements to specify the selected behavior. They do not broaden the six-finding delivery scope or approve implementation; the separate Routing Plan and execution gate remain required.
