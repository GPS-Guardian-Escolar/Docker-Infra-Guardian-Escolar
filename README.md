# Docker Infra

Esta carpeta centraliza la ejecucion de Guardian Escolar con Docker Compose.

## Credenciales de base de datos

Liquibase se conecta con `POSTGRES_USER` y `POSTGRES_PASSWORD`. El backend utiliza el rol de minimo privilegio `gps_guardian_app` y `APP_DB_PASSWORD`; no utiliza la cuenta superusuario de PostgreSQL.

Define `APP_DB_PASSWORD` en el archivo local `.env` antes de iniciar el stack. Debe tener al menos 32 caracteres compuestos por letras, numeros, guion o guion bajo. No guardes secretos reales en Git.

El rol de aplicacion debe estar habilitado por las migraciones de DB-Structure antes de iniciar el backend. La base `dev` actual crea el rol como `NOLOGIN`; integra primero la migracion de DB-Structure que habilita el login.

Compose usa el contexto Liquibase `prod` por defecto. Para ejecutar la carga administrativa local, cambia `LIQUIBASE_CONTEXTS=local` y define `ADMIN_EMAIL`, `ADMIN_PHONE` y un `ADMIN_PASSWORD_HASH` BCrypt.

## Ejecutar desde esta carpeta

Copia `.env.example` a `.env`, completa los secretos y ejecuta:

~~~powershell
docker compose --env-file .\.env -f .\docker-compose.yaml config
docker compose --env-file .\.env -f .\docker-compose.yaml up --build
~~~

El backend se conecta al puerto 5432 interno de PostgreSQL por medio del servicio `database`.
