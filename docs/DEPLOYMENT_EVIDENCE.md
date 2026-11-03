# Deployment evidence

The `Infrastructure quality` workflow builds the four-repository stack, starts it with
Docker Compose, and verifies both public health endpoints:

- Backend: `GET http://127.0.0.1:8080/api/health`
- Frontend: `GET http://127.0.0.1:8081/`

Every run uploads the Compose status, rendered configuration, container logs, and the
rollback guide as the `deployment-evidence` artifact. PostgreSQL is version-pinned to
`postgres:15.8-alpine` for reproducible builds.
