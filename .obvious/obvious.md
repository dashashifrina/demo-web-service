# Repo guidance — dashashifrina/demo-web-service

Simple Go REST API for user management (stdlib `net/http`, no framework, in-memory data).

## Stack

- **Language:** Go 1.24 (module `github.com/kakkoyun/demo-web-service`)
- **App:** single HTTP API service — entry point `cmd/api/main.go`, listens on `$SERVER_PORT` (default **8080**)
- **External services:** none required — all data is in-memory; no DB, no Redis, no queue
- **Observability:** optional Datadog tracing via `make dev` (orchestrion toolexec + `DD_*` env vars); omit for plain dev
- **Load-test infra (optional):** `tests/docker-compose.yml` (InfluxDB 8086, Grafana 3000) — only for k6 load testing, not needed for local dev

## Commands

| Task | Command | Notes |
|---|---|---|
| Build | `go build -o demo-web-service ./cmd/api` | plain build, no extra toolchain |
| Run (dev) | `go run ./cmd/api` | simplest local dev; port 8080 |
| Run (traced) | `make dev` | requires `DD_*` env vars + orchestrion |
| Test (all) | `go test ./...` | unit in `handlers/`, integration in `tests/` |
| Test (scoped) | `go test ./handlers/... -v` | fast loop for handler changes |
| Vet | `go vet ./...` | |
| Lint | `go tool golangci-lint run ./...` | golangci-lint is a Go tool dep |
| Lint (all) | `make lint` | Go + shellcheck (shellcheck not in sandbox) |
| Format | `make fmt` | go + shfmt + yamlfmt |
| README sync | `make embedmd-check` | CI fails if README snippets drift |

## Environment

No secrets required. All env vars have safe defaults:

| Variable | Default | Purpose |
|---|---|---|
| `SERVER_PORT` | `8080` | HTTP listen port |
| `READ_TIMEOUT` / `WRITE_TIMEOUT` / `IDLE_TIMEOUT` | `15s` / `15s` / `60s` | HTTP server timeouts |
| `ALLOWED_ORIGINS` | `http://localhost:3000,http://localhost:8080` | CORS allowed origins |
| `APP_ENV` | (unset) | `production` switches slog to Info level |

## API

| Method | Path | Notes |
|---|---|---|
| GET | `/` | welcome message |
| GET | `/api/health` | `{"status":"healthy"}` |
| GET | `/api/version` | build info |
| GET | `/api/users` | seeded list (John Doe, Jane Smith) |
| POST | `/api/users` | 201 on success |
| GET | `/api/users/{id}` | 400 on non-numeric id |
| DELETE | `/api/users/{id}` | 200 on success |

**Important:** handlers inject random errors outside test mode (`GET /` 10%, `GET /api/users` 20%, `POST /api/users` ~58% combined, `GET /api/users/{id>10}` 50%). Retry requests when verifying manually; tests set `handlers.TestMode = true`.

## Codebase Map

See `.obvious/codebase-map.md`.

## Rules

- Do not edit `orchestrion.tool.go` — auto-managed by `github.com/DataDog/orchestrion`.
- No `.env` files — configure via shell env vars only (`.env` is gitignored).
- README embeds code snippets via embedmd — run `make embedmd` after changing quoted funcs, or CI (`make embedmd-check`) fails.
- Go 1.22+ pattern routing (`mux.HandleFunc("GET /api/users/{id}", ...)`) — `r.PathValue("id")` for params.

## Local Verification Summary

> **Warning:** full-repo lint/test may be slow on cold caches. Use scoped commands for fast loops.

**Run date:** 2026-08-25 · **Dev stack healthy:** yes

| Check | Command | Result |
|---|---|---|
| Build | `go build -o demo-web-service ./cmd/api` | ✅ pass |
| Vet | `go vet ./...` | ✅ pass |
| Lint | `go tool golangci-lint run ./...` | ✅ pass (0 findings) |
| Tests | `go test ./...` | ✅ 6 pass / 0 fail (handlers 35.3% cov, tests pkg) |
| Server | `./demo-web-service` | ✅ up on :8080 |
| Health | `GET /api/health` | ✅ 200 `{"status":"healthy"}` |
| CRUD flow | list → create → get → delete | ✅ 200/201/200/200 with expected JSON |

### Scoped workflow

1. `go vet ./<pkg>/...`
2. `go tool golangci-lint run ./<pkg>/...`
3. `go test ./<pkg>/... -v`

## Sandbox Snapshot

- **Snapshot ID:** `vsjt5eabqlpc4exgjhid:default`
- **Captured:** `2026-08-25T19:27:01.278Z`
- **State:** Go 1.24.3 installed at `/usr/local/go` (add to PATH), modules cached, binary built, server verified healthy on :8080

## Runbooks

[Populated by autobuild-runbooks skill when requested. See `.obvious/runbooks/` after that skill runs.]
