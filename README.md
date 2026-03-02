# Keycloak

Docker stack to run `Keycloak + Postgres` behind `Traefik` (TLS via labels).

## Architecture
- `keycloak-db` (`postgres:16-alpine`): Keycloak database.
- `keycloak` (`quay.io/keycloak/keycloak:26.5`): IAM.
- External `traefik` network: communication with the reverse proxy.
- External `internal` network: private communication between app and database.

## Prerequisites
- Docker and Docker Compose installed.
- Traefik stack already running and connected to the `${TRAEFIK_NET}` network.
- Existing external networks:
```bash
docker network create traefik-net || true
docker network create internal-net || true
```

## Configuration
1. Copy the example file:
```bash
cp .env.example .env
```
2. Edit `.env` with real values:
- `TRAEFIK_NET`: name of Traefik's external network.
- `INTERNAL_NET`: internal network between services.
- `KC_HOSTNAME`: public Keycloak domain (example: `auth.yourdomain.com`).
- `POSTGRES_PASSWORD`: strong database password.
- `KEYCLOAK_ADMIN`: initial admin user.
- `KEYCLOAK_ADMIN_PASSWORD`: strong initial admin password.

3. Protect `.env` on the host:
```bash
chmod 600 .env
```

## Start the project
```bash
docker compose up -d
```

## Check status
```bash
docker ps --format 'table {{.Names}}\t{{.Status}}' | rg 'keycloak|keycloak-db'
```

Expected:
- `keycloak-db`: `healthy`
- `keycloak`: `healthy`

## Healthchecks
- `postgres`: `pg_isready -U keycloak -d keycloak`
- `keycloak`: local TCP check on port `9000` (management/health)

## Logs
```bash
docker logs -f keycloak
docker logs -f keycloak-db
```

## Useful commands
- Recreate stack:
```bash
docker compose up -d --force-recreate
```
- Stop stack:
```bash
docker compose down
```
- Stop stack and remove volumes:
```bash
docker compose down -v
```

## Quick troubleshooting
- `keycloak` stuck in `starting` for too long: validate connectivity with `postgres` and `POSTGRES_PASSWORD` variables.
- Routing/HTTPS error: verify `keycloak` labels and whether Traefik is on `${TRAEFIK_NET}`.
- Admin not created: check `KEYCLOAK_ADMIN` and `KEYCLOAK_ADMIN_PASSWORD` in `.env`.
