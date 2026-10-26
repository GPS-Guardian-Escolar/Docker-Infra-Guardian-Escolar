# Docker Infra

Esta carpeta centraliza la ejecucion por contenedores de GPS Guardian Escolar. Los cuatro repositorios deben estar clonados como carpetas hermanas dentro del mismo directorio:

- `DB-Structure-Guardian-Escolar`
- `Backend-Guardian-Escolar`
- `Fronted-GuardianEscolar-`
- `Docker-Infra-Guardian-Escolar`

## Preparacion

Desde esta carpeta, copia `.env.example` a `.env` y completa las variables privadas. No uses el `.env` de otro repositorio ni guardes contrasenas reales en Git.

## Ejecutar desde esta carpeta

```powershell
docker compose --env-file .\.env -f .\docker-compose.yaml config
docker compose --env-file .\.env -f .\docker-compose.yaml up --build
```

El primer comando valida la configuracion antes de arrancar. Para ejecutar desde la raiz que contiene los cuatro repositorios:

```powershell
docker compose --env-file .\Docker-Infra-Guardian-Escolar\.env -f .\Docker-Infra-Guardian-Escolar\docker-compose.yaml up --build
```

## Servicios y puertos

- Frontend: `http://localhost:8081`
- Backend: `http://localhost:8080`
- Base de datos: `localhost:5432` (solo desarrollo local)

La base de datos usa el volumen `gps_guardian_escolar_postgres_data`. `docker compose down` conserva los datos; `docker compose down -v` los elimina.

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
