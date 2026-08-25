# Codebase Map

| Directory | Purpose |
|---|---|
| `cmd/api` | Application entry point — HTTP server setup, routing, middleware wiring, graceful shutdown |
| `config` | Env-based configuration loading (`SERVER_PORT`, timeouts, CORS origins) |
| `handlers` | HTTP handlers — home, health, version, user CRUD; logging middleware; unit tests |
| `models` | Data models — `User`, `UserResponse` |
| `tests` | Integration tests (`api_test.go`), k6 load tests, Docker Compose (InfluxDB + Grafana) |
| `tests/grafana` | Grafana provisioning — k6 dashboard + InfluxDB datasource |
| `tools` | `lint-all.sh` — runs all linters/formatters (optional `--fix`) |
| `.github/workflows` | CI/CD — test+coverage, verify (lint/fmt/embedmd), build, release, security scans |
