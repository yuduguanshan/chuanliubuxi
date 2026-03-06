# AGENTS.md

## Cursor Cloud specific instructions

This is a simple Go 1.21+ microservice using the Gin web framework. It is a stateless HTTP API with no database or external service dependencies.

### Quick Reference

| Task | Command |
|---|---|
| Install deps | `go mod download` |
| Build | `go build -o server .` |
| Lint | `go vet ./...` |
| Run (dev) | `go run main.go` (listens on `:8080`) |

### Endpoints

- `GET /ping` — returns `{"message":"pong"}`
- `GET /hello/:name` — returns `{"message":"Hello, <name>"}`
- `POST /echo` — accepts JSON body and echoes it back

### Notes

- There are no automated tests in this repo. Validation is done via `go vet` and manual endpoint testing with `curl`.
- The built binary `server` is gitignored by convention; do not commit it.
- Kubernetes and ArgoCD manifests under `k8s/` and `argocd/` are for deployment only and not needed for local development.
