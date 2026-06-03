---
name: local-dev
version: 1.0.0
description: Bring this repo local development environment up from scratch.
category: local-dev
triggers:
  - local dev setup
  - run repo locally
  - start dev server
  - bring up local stack
author: autobuild-setup
created: 2026-06-03
---

## Prerequisites

- **Go 1.24+** required. Not pre-installed in the sandbox. Install via:

      curl -fsSL https://go.dev/dl/go1.24.3.linux-amd64.tar.gz -o /tmp/go.tar.gz
      tar -C /usr/local -xzf /tmp/go.tar.gz
      export PATH=$PATH:/usr/local/go/bin

- No databases, no Redis, no external services — the service is self-contained with in-memory data.

## Install

    export PATH=$PATH:/usr/local/go/bin
    go mod download
    go build -o demo-web-service ./cmd/api

## Environment

No required secrets. Optional Datadog env vars (omit for plain local dev):

| Variable | Default | Purpose |
|---|---|---|
| SERVER_PORT | 8080 | HTTP listen port |
| READ_TIMEOUT | 15s | HTTP read timeout |
| WRITE_TIMEOUT | 15s | HTTP write timeout |
| IDLE_TIMEOUT | 60s | HTTP idle timeout |
| ALLOWED_ORIGINS | http://localhost:3000,http://localhost:8080 | CORS allowed origins |
| DD_ENV | (optional) | Datadog environment tag |
| DD_SERVICE | (optional) | Datadog service name |

No .env file needed - all defaults are safe for local use.

## Start

    nohup ./demo-web-service > /tmp/demo-web-service.log 2>&1 &
    # Actual port observed: 8080

Or for development without background:

    go run ./cmd/api

## Verify Primary User Flow

    # Health check
    curl http://localhost:8080/api/health
    # Expected: {"status":"healthy"}

    # List users (seeded in-memory data)
    curl http://localhost:8080/api/users
    # Expected: {"status":"success","users":[{"name":"John Doe","id":1},{"name":"Jane Smith","id":2}]}

    # Get user by ID
    curl http://localhost:8080/api/users/1
    # Expected: {"status":"success","user":{"name":"User 1","id":1}}

    # Home
    curl http://localhost:8080/
    # Expected: {"message":"Welcome to the API"}

## Verified Commands

- Build: go build -o demo-web-service ./cmd/api
- Vet: go vet ./...
- Test: go test ./... (9 tests: handlers + integration - all PASS)
- Scoped test: go test ./handlers/... -v

## Sandbox Snapshot

- snapshotId: gbvrpg28ron5icojoq21:default - restoring reproduces verified-healthy state.

## Known Blockers / Workarounds

- **Go not in PATH:** Install manually (see Prerequisites). Required Go 1.24+.
- **Port 8080 in use:** Find process with: ss -tlnp sport = :8080 and kill by PID.
- **make dev requires Datadog env vars:** Uses orchestrion toolexec + DD_* env vars. Use go run ./cmd/api for plain local dev.
