# Docker Infra

Esta carpeta centraliza la ejecucion por contenedores de Guardian Escolar.

## Despliegue reproducible

Antes de publicar una version, fija cada repositorio en el commit aprobado y registra los SHA de los cuatro repositorios junto con las referencias completas de sus imagenes. Para Liquibase, Backend y Frontend usa nombres de imagen de un registry y etiquetas inmutables derivadas del release. Fija tambien PostgreSQL por digest para produccion.

Define `POSTGRES_IMAGE`, `LIQUIBASE_IMAGE`, `LIQUIBASE_IMAGE_TAG`, `BACKEND_IMAGE`, `BACKEND_IMAGE_TAG`, `FRONTEND_IMAGE` y `FRONTEND_IMAGE_TAG` en el archivo de release. Los valores locales de `.env.example` sirven para desarrollo y no son referencias de release.

La base de datos conserva `POSTGRES_USER` y `POSTGRES_PASSWORD` únicamente como
credenciales de bootstrap para PostgreSQL y Liquibase. El contenedor `backend` se conecta
con `APP_DB_USERNAME` (por defecto `gps_guardian_app`) y `APP_DB_PASSWORD`, el rol de
mínimo privilegio creado por los changesets de `DB-Structure-Guardian-Escolar`. No se
deben copiar las credenciales del superusuario al servicio de aplicación.

Para comprobarlo en un entorno local después del arranque:

~~~powershell
docker compose --env-file .\.env exec database psql -U gps_guardian_app -d gps_guardian_escolar -c "select current_user;"
~~~

El resultado esperado es `gps_guardian_app`, nunca `postgres`. `APP_DB_PASSWORD` debe
ser un secreto local/de despliegue y no se publica en este repositorio.

Construye y publica solo las imagenes propias:

~~~powershell
docker compose --env-file .\.env build --pull liquibase backend frontend
docker compose --env-file .\.env push liquibase backend frontend
~~~

En el host de despliegue, configura el mismo archivo de release y sus secretos, descarga esas referencias e inicia el stack sin reconstruir:

~~~powershell
docker compose --env-file .\.env pull
docker compose --env-file .\.env up -d
~~~

El servicio Liquibase debe finalizar antes de iniciar el Backend. Comprueba salud de la base, el endpoint de salud del Backend y el acceso al Frontend antes de aceptar trafico.

## Rollback

Restaura en el manifiesto las referencias inmutables de la version aprobada anterior, luego descarga y vuelve a iniciar las imagenes:

~~~powershell
docker compose --env-file .\.env pull
docker compose --env-file .\.env up --no-build -d backend frontend
~~~

Este paso revierte las imagenes de aplicacion; no revierte cambios de esquema. Ejecuta un rollback Liquibase revisado solo despues de respaldar y validar la base en un entorno descartable. `docker compose down` conserva el volumen de PostgreSQL. No uses `docker compose down -v` como rollback.
