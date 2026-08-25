---
name: local-dev
version: 1.1.0
description: Bring this repo's local development environment up from scratch.
category: local-dev
triggers:
  - local dev setup
  - run repo locally
  - start dev server
  - bring up local stack
author: autobuild-setup
created: 2026-06-03
updated: 2026-08-25
---

## Prerequisites

- **Go 1.24+** — not pre-installed in a fresh sandbox. Install:

      curl -fsSL https://go.dev/dl/go1.24.3.linux-amd64.tar.gz -o /tmp/go.tar.gz
      tar -C /usr/local -xzf /tmp/go.tar.gz
      export PATH=$PATH:/usr/local/go/bin

- No databases, no Redis, no Docker needed — the service is self-contained with in-memory data.
- `tests/docker-compose.yml` (InfluxDB/Grafana) is only for optional k6 load testing.

## Install

    export PATH=$PATH:/usr/local/go/bin
    go mod download
    go build -o demo-web-service ./cmd/api

Note: `go mod download` may print a transient `proxy.golang.org ... INTERNAL_ERROR` for an
indirect tool dep (e.g. `aws-sdk-go-v2/internal/v4a`) — the main build still succeeds.

## Environment

No secrets required. All vars have safe defaults:

| Variable | Default | Purpose |
|---|---|---|
| SERVER_PORT | 8080 | HTTP listen port |
| READ_TIMEOUT | 15s | HTTP read timeout |
| WRITE_TIMEOUT | 15s | HTTP write timeout |
| IDLE_TIMEOUT | 60s | HTTP idle timeout |
| ALLOWED_ORIGINS | http://localhost:3000,http://localhost:8080 | CORS origins |
| APP_ENV | (unset) | `production` → Info-level logs |

## Start

    nohup ./demo-web-service > /tmp/demo-web-service.log 2>&1 &
    sleep 2 && grep -o '"serverPort":"[0-9]*"' /tmp/demo-web-service.log
    # Actual port observed: 8080 (also visible via ss -tlnp | grep 8080)

Foreground alternative: `go run ./cmd/api` (plain) or `make dev` (Datadog-traced, needs DD_* vars).

## Verify Primary User Flow

**Handlers inject random errors outside test mode — retry until the expected code.**

    curl -s http://localhost:8080/api/health
    # {"status":"healthy"}

    curl -s http://localhost:8080/api/users          # retry: 20% random 500
    # {"status":"success","users":[{"name":"John Doe","id":1},{"name":"Jane Smith","id":2}]}

    curl -s -X POST http://localhost:8080/api/users -H "Content-Type: application/json" -d '{"name":"New User"}'
    # 201 {"status":"success","message":"User created successfully","user":{"name":"New User","id":3}}

    curl -s http://localhost:8080/api/users/1
    # {"status":"success","user":{"name":"User 1","id":1}}

    curl -s -X DELETE http://localhost:8080/api/users/3
    # {"status":"success","message":"User deleted successfully"}

    curl -s http://localhost:8080/api/users/abc       # 400 invalid id

## Verified Commands (2026-08-25)

- Build: `go build -o demo-web-service ./cmd/api` — pass
- Vet: `go vet ./...` — pass
- Lint: `go tool golangci-lint run ./...` — pass, 0 findings
- Tests: `go test ./...` — 6 pass / 0 fail (handlers 35.3% coverage + integration pkg)
- Server: healthy on :8080, full CRUD flow verified

## Sandbox Snapshot

- snapshotId: `vsjt5eabqlpc4exgjhid:default` (captured 2026-08-25T19:27:01.278Z)
- Restoring reproduces the verified-healthy state: Go 1.24.3 installed, modules cached, server verified.

## Known Blockers / Workarounds

- **Go not in PATH:** install per Prerequisites; fresh sandboxes ship without Go.
- **Port 8080 in use:** `ss -tlnp | grep 8080`, kill stale PID, or set `SERVER_PORT`.
- **`make dev` needs Datadog env vars:** uses orchestrion toolexec + DD_* vars — use `go run ./cmd/api` for plain dev.
- **shellcheck missing in sandbox:** `make lint` fails on the shellcheck step; use `go tool golangci-lint run ./...` for Go lint evidence.
- **Random handler errors:** expected behavior outside `TestMode` — retry requests, don't file bugs.
