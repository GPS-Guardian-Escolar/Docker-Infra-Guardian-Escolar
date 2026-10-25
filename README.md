# Docker Infra

Esta carpeta centraliza la ejecucion por contenedores de GPS Guardian Escolar.

## Servicios

- `database`: PostgreSQL 15 con la estructura cargada desde `../DB-Structure-Guardian-Escolar`.
- `backend`: API Spring Boot construida desde `../Backend-Guardian-Escolar`.
- `frontend`: aplicacion Expo Web construida desde `../Fronted-GuardianEscolar-` y servida con Nginx.

## Preparacion

Clona los cuatro repositorios como carpetas hermanas y conserva exactamente estos nombres:

```text
ENTREGABLE 3.5/
  Docker-Infra-Guardian-Escolar/
  DB-Structure-Guardian-Escolar/
  Backend-Guardian-Escolar/
  Fronted-GuardianEscolar-/
```

Copia `.env.example` a `.env` y define como minimo `POSTGRES_PASSWORD` y `JWT_SECRET`.
La clave JWT debe ser aleatoria y tener al menos 32 bytes. No guardes `.env` en Git.

## Ejecutar desde la raiz del proyecto

```powershell
docker compose --env-file .\Docker-Infra-Guardian-Escolar\.env -f .\Docker-Infra-Guardian-Escolar\docker-compose.yaml up --build
```

## Ejecutar desde esta carpeta

```powershell
docker compose --env-file .\.env up --build
```

Para una validacion sin iniciar contenedores:

```powershell
docker compose --env-file .\.env config --quiet
```

## Puertos

- Frontend: `http://localhost:8081`
- Backend: `http://localhost:8080`
- Base de datos: `localhost:5432`

## Persistencia

La base de datos guarda los datos en el volumen:

```text
gps_guardian_escolar_postgres_data
```

Usa `docker compose down` para apagar conservando datos.
Usa `docker compose down -v` solo si quieres borrar tambien la base de datos.

## Despliegue y rollback

El flujo operativo esta documentado en [`docs/DEPLOYMENT_ROLLBACK.md`](docs/DEPLOYMENT_ROLLBACK.md).
Las migraciones se aplican mediante el servicio `liquibase` antes de iniciar el backend.
No uses `down -v` como rollback: elimina el volumen y los datos locales.

## Uso con contenedor o tunnel

Cuando el frontend se sirve desde el contenedor, usa rutas relativas para que el mismo dominio del navegador atienda frontend y API:

```env
EXPO_PUBLIC_API_URL=/api
EXPO_PUBLIC_WS_URL=/ws
EXPO_PUBLIC_STUDENT_LINK_BASE_URL=/
```

Nginx recibe `/api` y `/ws` en el contenedor frontend y los reenvia al servicio `backend` dentro de Docker. Esto permite abrir un tunnel solo al puerto `8081` del frontend.

## Uso con Expo Go o LAN

Si abres la app desde Expo Go o desde el servidor de desarrollo sin Nginx, usa una IP/URL alcanzable para el backend:

```env
EXPO_PUBLIC_API_URL=http://TU_IP:8080/api
EXPO_PUBLIC_WS_URL=ws://TU_IP:8080/ws
EXPO_PUBLIC_STUDENT_LINK_BASE_URL=http://TU_IP:8081
FRONTEND_BASE_URL=http://TU_IP:8081
API_PUBLIC_BASE_URL=http://TU_IP:8080
```
