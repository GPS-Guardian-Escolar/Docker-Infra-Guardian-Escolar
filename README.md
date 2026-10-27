# Docker Infra

Esta carpeta centraliza la ejecucion por contenedores de GPS Guardian Escolar.

## Servicios

- `database`: PostgreSQL 15 con la estructura cargada desde `../DB-Structure-Guardian-Escolar`.
- `backend`: API Spring Boot construida desde `../Backend-Guardian-Escolar`.
- `frontend`: aplicacion Expo Web construida desde `../Fronted-GuardianEscolar-` y servida con Nginx.

## Ejecutar desde la raiz del proyecto

```powershell
docker compose --env-file .\docker-infra\.env -f .\docker-infra\docker-compose.yaml up --build
```

## Ejecutar desde esta carpeta

```powershell
docker compose --env-file .\.env up --build
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
