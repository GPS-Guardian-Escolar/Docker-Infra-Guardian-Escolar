# Deployment and Rollback

## Release preflight

1. Record the approved commit SHA for Docker Infra, DB Structure, Backend and Frontend.
2. Set the release image references in `.env`. Use immutable registry tags for Liquibase, Backend and Frontend, and a digest-pinned PostgreSQL image in production.
3. Set non-empty `POSTGRES_PASSWORD` and a random `JWT_SECRET` of at least 32 bytes. When the least-privilege application-role change is part of the target release, also set the required `APP_DB_PASSWORD` before starting the stack.
4. For a first deployed administrator only, run Liquibase once with `LIQUIBASE_CONTEXTS=prod,bootstrap` and inject `ADMIN_EMAIL`, `ADMIN_PHONE` and a BCrypt `ADMIN_PASSWORD_HASH` from the environment secret store. Do not commit these values.
5. For normal deployments, use `LIQUIBASE_CONTEXTS=prod`; the bootstrap variables are not needed.
6. Validate the rendered Compose configuration:

~~~powershell
docker compose --env-file .\.env config --quiet
~~~

Keep a release manifest with the four repository SHAs, image references, deployment date and validation results. Do not store secret values in that manifest.

## Build and publish

Build the app images from the recorded repository commits and publish them to the configured registry:

~~~powershell
docker compose --env-file .\.env build --pull liquibase backend frontend
docker compose --env-file .\.env push liquibase backend frontend
~~~

Do not rebuild images on the deployment host. Use `docker compose pull` and `docker compose up -d` with the same release file. Liquibase must complete before Backend starts.

## Rollback

Restore the previous approved immutable image references in the release file, then run:

~~~powershell
docker compose --env-file .\.env pull
docker compose --env-file .\.env up --no-build -d backend frontend
~~~

This restores application images only. Database changesets require a separate reviewed Liquibase rollback after backup and validation in a disposable environment. Never use `docker compose down -v` as an application rollback; it deletes the PostgreSQL volume.
