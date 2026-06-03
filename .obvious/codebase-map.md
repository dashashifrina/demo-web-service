# Codebase Map

| Directory | Purpose |
|---|---|
| `cmd/api` | Application entry point — starts HTTP server, wires routes and config |
| `config` | Environment-based configuration loading (SERVER_PORT, timeouts, CORS) |
| `handlers` | HTTP request handlers — home, health check, user CRUD |
| `models` | Data models — User struct |
| `tests` | Integration tests and k6 load testing setup with Docker Compose |
| `tools` | Shell scripts for running linters |
| `.github/workflows` | CI/CD — lint, test, build, release, and security scan pipelines |
