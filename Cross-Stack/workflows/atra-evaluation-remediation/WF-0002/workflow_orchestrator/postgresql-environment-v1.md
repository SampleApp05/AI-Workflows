---
artifact:
  id: ENV-0001
  type: ENVIRONMENT_RECORD
  version: 1
  status: READY
workflow:
  id: WF-0002
  project: Atra
  technology: Cross-Stack
  feature: atra-evaluation-remediation
  mode: delivery
  contract_version: "1.2"
lineage:
  parents:
    - DEC-0003@v1
ownership:
  created_by: infrastructure_provisioning_worker
  performed_by: infrastructure_provisioning_worker
  recorded_by: workflow_orchestrator
  engine: claude-code
  target_id: claude-cli
---

# Disposable PostgreSQL Environment — WF-0002

## Verified product compatibility

Read-only inspection of `Atra-Services` (local checkout, `packages/database`) found:

- Database package `@atra/database` uses **Drizzle ORM** (`drizzle-orm ^0.45.2`) with dialect `postgresql` (`packages/database/drizzle.config.ts`).
- PostgreSQL driver is **postgres.js** (`postgres ^3.4.9`); the client factory `createDb(connectionString)` in `packages/database/src/client.ts` takes a caller-supplied connection string and does not hardcode a host.
- Migration tool is **drizzle-kit** (`drizzle-kit ^0.31.10`), invoked via `db:generate` / `db:migrate` / `db:studio` scripts, with migration output directory `packages/database/migrations` (currently contains only `.gitkeep` — no migrations have been generated in this checkout).
- The only documented local connection convention found in the repo is `apps/auth-service/.env.example`: `DATABASE_URL=postgres://postgres:postgres@localhost:5432/atra_dev` (role `postgres`, database `atra_dev`, default port `5432`). No `docker-compose` file or other PostgreSQL version pin exists in the repository.
- No PostgreSQL binaries were present on this host before provisioning (`postgres`, `psql`, `pg_ctl`, `initdb` all unresolved via `which`).

No migration was generated or applied, and no schema was loaded into the provisioned database — that is out of scope for this artifact.

## Isolation and guardrails

- Homebrew installed the `postgresql@17` formula (17.11) to provide binaries; this was the only host package change. The formula's own post-install step created its default cluster at `/opt/homebrew/var/postgresql@17` as a side effect of installation — that cluster was **not started** (`brew services start` was never run) and is not used by this environment.
- All mutable database state lives under the exact disposable path `/tmp/atra-wf0002-postgres` (data directory, unix-socket directory, and server log). The path was verified empty immediately before `initdb`. The only intentional modification to the host outside this path or this artifact was the authorized Homebrew installation of `postgresql@17` described above, including its standard post-install default cluster at `/opt/homebrew/var/postgresql@17`; that default cluster was not started or used.
- The server was started directly with `pg_ctl` against `/tmp/atra-wf0002-postgres/data`, not as a Homebrew service, and is not configured to persist across host reboots.
- `postgresql.conf` sets `listen_addresses = '127.0.0.1'` and a dedicated `unix_socket_directories = '/tmp/atra-wf0002-postgres/sockets'`. Confirmed via `lsof -nP -iTCP:55432 -sTCP:LISTEN`: a single listener bound to `127.0.0.1:55432` only (no `0.0.0.0` or external interface).
- `pg_hba.conf` (initdb default) permits only the local unix socket and `127.0.0.1/32` / `::1/128` — no non-local host entries.
- A dedicated, non-privileged role and database named `atra_wf0002` were created (`initdb -U atra_wf0002`, `createdb -O atra_wf0002 atra_wf0002`) — distinct from the repo-documented `postgres`/`atra_dev` defaults and from any shared/global role.
- No production endpoint, shared database, or existing data directory was touched. No product-repository file was changed.

## Non-secret connection reference

| Field | Value |
|---|---|
| Host | `127.0.0.1` |
| Port | `55432` |
| Database | `atra_wf0002` |
| Role | `atra_wf0002` |
| Auth method | `trust`, loopback/unix-socket only (no password exists or is required) |
| Unix socket dir | `/tmp/atra-wf0002-postgres/sockets` |
| Example URL | `postgres://atra_wf0002@127.0.0.1:55432/atra_wf0002` |

No credential is embedded here because `trust` authentication over a loopback-only, filesystem-permission-protected socket/port requires none.

## Liveness evidence

Non-destructive checks performed against the running instance (raw output kept out of this artifact):

- `pg_isready -h 127.0.0.1 -p 55432` → accepting connections.
- `SELECT version()` → PostgreSQL 17.11 (Homebrew) on `aarch64-apple-darwin25.6.0`.
- `SELECT current_user, current_database(), inet_server_addr(), inet_server_port()` → `atra_wf0002` / `atra_wf0002` / `127.0.0.1` / `55432`.
- `lsof -nP -iTCP:55432 -sTCP:LISTEN` → exactly one listener, bound to `127.0.0.1:55432`.

## Permitted future use

This environment may be used, within this workflow's later approved stages, for:

- Applying the WF-0002 PostgreSQL migration once execution is approved.
- Running the read-only incompatibility preflight.
- Capturing competing-write evidence.

It is not authorized for production data, any use outside WF-0002, or persistence beyond this workflow's disposable lifecycle.

## Representative-preflight limitation

This is a freshly initialized, empty PostgreSQL instance with no schema loaded and no data population. It verifies only that a compatible PostgreSQL engine and driver/ORM combination can run locally — it cannot and does not establish anything about an existing data population (no duplicate-role, owner, recovery, or canonical-owner-inconsistency evidence is possible against an empty database). Before the read-only incompatibility preflight can be treated as representative, the workflow still needs one of:

1. A credential-safe, human-confirmed sanitized representative dataset or source to load into this environment, or
2. An explicit human statement that no existing applicable data population exists.

No such dataset or statement is created, seeded, or assumed by this provisioning step.

## Teardown

Operator command to fully remove this disposable environment when no longer needed:

```sh
/opt/homebrew/opt/postgresql@17/bin/pg_ctl -D /tmp/atra-wf0002-postgres/data stop -m fast
rm -rf /tmp/atra-wf0002-postgres
```

This stops only the server rooted at the exact isolated data directory above, then removes only that exact path. No other process or data directory is affected.

## Outcome

Status: **READY**. Local provisioning succeeded: an isolated PostgreSQL 17.11 instance is running, bound to `127.0.0.1:55432` only, backed entirely by `/tmp/atra-wf0002-postgres`, with a dedicated `atra_wf0002` role/database and passing liveness checks. Environment readiness is established; representative-data readiness for the incompatibility preflight is not, and remains blocked on the human-provided evidence described above.
