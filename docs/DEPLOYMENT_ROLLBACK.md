# Deployment and Rollback

## Preflight

1. Confirm the four repositories are sibling directories with the names listed in the infrastructure README.
2. Create `.env` from `.env.example` and provide non-empty `POSTGRES_PASSWORD` and a random `JWT_SECRET` of at least 32 bytes.
3. Validate the rendered compose file without starting services:

   ```powershell
   docker compose --env-file .\.env config --quiet
   ```

4. Review the target Git commits for the infrastructure, database and backend repositories before deployment.

## Deployment

Build and start the stack from this repository:

```powershell
docker compose --env-file .\.env up --build -d
docker compose --env-file .\.env ps
```

The `database` healthcheck must pass first. Liquibase then runs `update`; the backend starts only after Liquibase completes successfully. Confirm the backend health endpoint and the frontend through the published port before accepting traffic.

## Rollback

1. Stop the application containers without deleting the PostgreSQL volume:

   ```powershell
   docker compose --env-file .\.env down
   ```

2. Restore the previous immutable `BACKEND_IMAGE_TAG` and `FRONTEND_IMAGE_TAG` values recorded for the approved release, then run `docker compose --env-file .\.env up --no-build -d backend frontend`. If an image must be rebuilt from source, check out the recorded commit SHA first and use the same release tag.
3. If a database changeset must be reverted, execute its reviewed Liquibase rollback from the database repository against a backup or disposable environment first. Do not edit an already applied changeset.
4. Start the stack again and verify health, authentication and the affected flow.

Never use `docker compose down -v` during an application rollback. That command deletes the local PostgreSQL volume and is reserved for intentional data destruction in disposable environments.
