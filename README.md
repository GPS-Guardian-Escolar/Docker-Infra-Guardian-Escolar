# Docker Infra

Esta carpeta centraliza la ejecucion por contenedores de GPS Guardian Escolar. Los cuatro repositorios deben estar clonados como carpetas hermanas dentro del mismo directorio:

- `DB-Structure-Guardian-Escolar`
- `Backend-Guardian-Escolar`
- `Fronted-GuardianEscolar-`
- `Docker-Infra-Guardian-Escolar`

## Servicios

- `database`: PostgreSQL.
- `liquibase`: ejecuta las migraciones de DB-Structure.
- `backend`: API Spring Boot con la cuenta de minimo privilegio `gps_guardian_app`.
- `frontend`: aplicacion Expo Web servida por Nginx.

## Preparacion

Clona los cuatro repositorios como carpetas hermanas y conserva exactamente estos nombres:

```text
ENTREGABLE 3.5/
  Docker-Infra-Guardian-Escolar/
  DB-Structure-Guardian-Escolar/
  Backend-Guardian-Escolar/
  Fronted-GuardianEscolar-/
```

Desde esta carpeta, copia `.env.example` a `.env` y completa las variables privadas. No uses el
`.env` de otro repositorio ni guardes contrasenas reales en Git. Configura `POSTGRES_PASSWORD`,
`APP_DB_PASSWORD` y `JWT_SECRET`. La clave JWT debe ser aleatoria y tener al menos 32 bytes.

Completa `POSTGRES_PASSWORD`, `APP_DB_PASSWORD` y `JWT_SECRET` antes de iniciar.

Configura `APP_DB_PASSWORD` con al menos 32 caracteres alfanumericos, guion o guion bajo.
Ese secreto habilita la cuenta de minimo privilegio `gps_guardian_app`; el backend no se conecta
como superusuario. Para despliegues sin seed local, Compose usa el contexto Liquibase `prod`.
En desarrollo local, `.env.example` opta por `LIQUIBASE_CONTEXTS=local`: define tambien
`ADMIN_EMAIL`, `ADMIN_PHONE` y un `ADMIN_PASSWORD_HASH` BCrypt antes de iniciar los servicios.

## Ejecutar desde esta carpeta

```powershell
docker compose --env-file .\.env -f .\docker-compose.yaml config
docker compose --env-file .\.env -f .\docker-compose.yaml up --build
```

Para una validacion sin iniciar contenedores:

```powershell
docker compose --env-file .\.env config --quiet
```

Para ejecutar desde la raiz que contiene los cuatro repositorios:

```powershell
docker compose --env-file .\Docker-Infra-Guardian-Escolar\.env -f .\Docker-Infra-Guardian-Escolar\docker-compose.yaml up --build
```

## Servicios y puertos

- Frontend: `http://localhost:8081`
- Backend: `http://localhost:8080`
- Base de datos: `localhost:5432` (solo desarrollo local)

La base de datos usa el volumen `gps_guardian_escolar_postgres_data`. `docker compose down` conserva los datos; `docker compose down -v` los elimina.

## Despliegue y rollback

El flujo operativo esta documentado en [`docs/DEPLOYMENT_ROLLBACK.md`](docs/DEPLOYMENT_ROLLBACK.md).
Las migraciones se aplican mediante el servicio `liquibase` antes de iniciar el backend.
No uses `down -v` como rollback: elimina el volumen y los datos locales.

## Etiquetas de imagen

Compose acepta `POSTGRES_IMAGE` y un nombre mas etiqueta para Liquibase, backend y frontend.
Los valores `local` del ejemplo se construyen desde los repositorios hermanos. Para un release,
usa una etiqueta inmutable (por ejemplo, el SHA del commit aprobado) y registra los SHA de los
cuatro repositorios junto con las etiquetas. No uses `latest` para produccion. Para volver a una
version de aplicacion, restaura las etiquetas aprobadas anteriores y ejecuta `docker compose up
--no-build -d backend frontend`; los cambios de esquema requieren el rollback Liquibase revisado
por separado.

## Uso con contenedor o tunnel

Cuando el frontend se sirve desde el contenedor, usa rutas relativas para que el mismo dominio atienda frontend y API:

```env
EXPO_PUBLIC_API_URL=/api
EXPO_PUBLIC_WS_URL=/ws
EXPO_PUBLIC_STUDENT_LINK_BASE_URL=/
```

Nginx recibe `/api` y `/ws` en el frontend y los reenvia al servicio `backend` dentro de Docker. Para Expo Go o LAN, usa una IP alcanzable:

```env
EXPO_PUBLIC_API_URL=http://TU_IP:8080/api
EXPO_PUBLIC_WS_URL=ws://TU_IP:8080/ws
EXPO_PUBLIC_STUDENT_LINK_BASE_URL=http://TU_IP:8081
FRONTEND_BASE_URL=http://TU_IP:8081
API_PUBLIC_BASE_URL=http://TU_IP:8080
```
