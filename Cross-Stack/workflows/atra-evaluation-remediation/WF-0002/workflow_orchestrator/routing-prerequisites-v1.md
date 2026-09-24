---
artifact:
  id: DEC-0003
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
    - ROUTE-0002@v1
ownership:
  created_by: workflow_orchestrator
  performed_by: human
  recorded_by: workflow_orchestrator
  engine: codex
  target_id: codex-native
---

# Routing Prerequisite Decisions — Atra evaluation remediation

## Source

The requesting human provided these directions in this task on 2026-09-24: make the product branch, define the PostgreSQL environment requirements, and instruct Claude to set it up with the access needed.

## Decisions and verified result

1. **Dedicated product head:** the controller is authorized to create `workflow/WF-0002-atra-evaluation-remediation` from the clean Atra-Services `main` baseline `be51242593fe6643aff11c491d4f8a6c0835aed5`. The local and `origin` branch were created at that commit; no product file changed and no product pull request was created.
2. **Provisioning authority:** Claude may use the local host tools needed to provision an isolated, disposable, non-production PostgreSQL environment. This does not authorize a production endpoint, production data access or operation, deployment, merge, release, shared persistent service, or product-source change.
3. **Minimum environment definition:** the environment must be loopback-only, isolated in `/tmp/atra-wf0002-postgres`, use a dedicated `atra_wf0002` role/database on an unused high local port, expose only a non-secret connection reference, provide liveness evidence and a teardown command, and be used only later for the approved migrations, read-only preflight, and competing-write evidence.
4. **Representativeness constraint:** local environment readiness does not itself prove compatibility of an existing population. Before the read-only incompatibility preflight may be accepted as representative, the workflow needs a credential-safe, human-confirmed sanitized representative dataset/source or a human statement that no existing applicable data population exists. No such evidence is created or assumed by provisioning.

## Effect

These decisions resolve the dedicated-branch routing prerequisite and authorize bounded environment provisioning. `ROUTE-0002@v1` remains blocked until the provisioning artifact is verified and the representative-preflight condition is resolved. A same-stage routing revision and validation remain required before the separate execution gate.
