# Repo guidance

## Codebase Map

See `.obvious/codebase-map.md`.

## Rules

<!-- synthesized from: README.md, Makefile, cmd/api/main.go, handlers/ (files found in SCAN) — agent-relevant rules only -->

- **Language:** Go 1.24+ (stdlib HTTP server, no framework)
- **Entry point:** `cmd/api/main.go` — starts the server on `$SERVER_PORT` (default 8080)
- **Build:** `go build -o demo-web-service ./cmd/api` — no toolchain beyond Go required for plain builds
- **Dev command:** `make dev` uses Datadog orchestrion toolexec — requires `DD_*` env vars. Use `go run ./cmd/api` for simple local dev
- **Tests:** `go test ./...` — unit tests in `handlers/`, integration tests in `tests/`
- **Lint:** `go tool golangci-lint run ./...` (requires `golangci-lint` as a Go tool dep); `go vet ./...` always available
- **No external services required** — all data is in-memory; no DB, no Redis, no queue
- **Configuration via env vars only** — `SERVER_PORT`, `READ_TIMEOUT`, `WRITE_TIMEOUT`, `IDLE_TIMEOUT`, `ALLOWED_ORIGINS`
- **No `.env` files** — configure via shell env or OS-level env vars
- **Datadog tracing** optional — set `DD_ENV`, `DD_SERVICE`, `DD_VERSION` etc. for APM; omit for plain local dev
- **Deliberate flakiness:** handlers return random errors in non-test mode (home 10% 503, list users 20% 500, create user ~50% 4xx) — retries are expected behavior, not bugs
- **Do not edit orchestrion-generated files** (`orchestrion.tool.go`) — auto-managed by `github.com/DataDog/orchestrion`

## Local Verification

> **Warning:** Running full-repo typecheck, lint, or tests may OOM or timeout in the sandbox for large repos.
> Use the scoped commands below when verifying changes.

### Verified Commands

- **Build:** `go build -o demo-web-service ./cmd/api` — verified ✅
- **Vet:** `go vet ./...` — verified ✅
- **Test:** `go test ./...` — verified ✅ (6 test funcs pass: handlers unit + tests integration, incl. new DELETE tests)
- **Lint:** `go tool golangci-lint run ./...` — available (requires Go tools installed)

### Primary Flow (verified live, 2026-10-30)

Server on port 8080 (parsed from startup log; default from `SERVER_PORT`):

1. `GET /api/health` → 200 `{"status":"healthy"}` ✅
2. `GET /` → 200 `{"message":"Welcome to the API"}` ✅
3. `GET /api/users` → 200 with in-memory users (John Doe, Jane Smith) ✅
4. `GET /api/users/1` → 200 `{"status":"success","user":{...}}` ✅
5. `POST /api/users` with JSON body → 201 `{"status":"success","message":"User created successfully",...}` ✅
6. `GET /api/users/notanumber` → 400 error JSON ✅
7. `DELETE /api/users/2` → 200 `{"status":"success","message":"User deleted successfully"}` ✅ (endpoint added after last contract refresh)
8. `DELETE /api/users/-1` → 400 error JSON ✅

### Scoped Workflow

Run these commands to verify changed files without triggering a full-repo scan:

1. **Typecheck changed files:** `go vet ./handlers/...` (or the relevant package)
2. **Lint changed files:** `go tool golangci-lint run ./handlers/...`
3. **Test changed files:** `go test ./handlers/... -v` or `go test ./tests/... -v`

## Sandbox Snapshot

- **Snapshot ID:** `ybub73g0f9juy3nraz0d:default`
- **Captured:** `2026-10-30T15:22:27.070Z`
- **Dev stack healthy:** yes

## Bibliography

- **Nodes upserted:** 4 (all new) — initial onboarding run
- **Slugs:** `demo-web-service`, `demo-web-service-handlers`, `demo-web-service-config`, `demo-web-service-datadog-tracing`
- **Root node:** `bib_mXWozNCD` (Demo Web Service — system)

## Security Scan

> **Note:** security_scan_not_triggered — repository `dashashifrina/demo-web-service` is not found in the workspace autobuild registry. Trigger manually using the trigger_security_onboarding tool with commit SHA `dae7f11953831bb90f909d6e6ac75b7f39a2d9cd` once the repo is connected via Settings → Repositories.

## Runbooks

[Populated by autobuild-runbooks skill when requested. See `.obvious/runbooks/` after that skill runs.]
