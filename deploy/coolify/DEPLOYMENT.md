# Coolify deployment (this fork)

This fork carries the upstream Dograh stack plus one deployment-specific change.

## What runs

Coolify application `dograh` (uuid `nu5ytc8d1h4vqid2ajwkmtmf`), project `Dograh`,
environment `production`, server `dev-dmytro` (10.11.12.15).

Compose file: `deploy/hostinger/docker-compose.yaml` — upstream's own variant for
platforms that already run a managed Traefik (see `deploy/hostinger/README.md`).
It is used verbatim; the only delta against upstream is the `expose:` lines added
below (upstream's canonical `docker-compose.yaml` declares the same ports).

    postgres  pgvector/pgvector:pg17   (internal network only)
    redis     redis:7                   (internal network only)
    minio     dograhai/minio:2026.09.22 (internal network only)
    coturn    coturn/coturn:4.8.0       (publishes the TURN ports — UDP cannot be proxied)
    api       dograhai/dograh-api:latest   expose 8000
    ui        dograhai/dograh-ui:latest    expose 3010

## Changes against upstream

1. `expose:` for api/ui/minio — the upstream canonical compose declares the same
   ports; without them the platform cannot infer the backend port for the router.
2. `deploy/coolify/DEPLOYMENT.md` (this file).

Everything else — the compose body, the service set, the env contract in
`deploy/hostinger/.env.example`, the TURN setup — is upstream's.

## Environment (the `.env` equivalent, kept in Infisical `/dograh`)

    PUBLIC_HOST=dograh.idstar.de
    TRAEFIK_NETWORK=coolify
    TRAEFIK_ENTRYPOINT=https
    TRAEFIK_CERTRESOLVER=letsencrypt
    TURN_HOST=78.94.156.46
    TURN_SECRET / OSS_JWT_SECRET / POSTGRES_PASSWORD / REDIS_PASSWORD /
    MINIO_ROOT_USER / MINIO_ROOT_PASSWORD   (Infisical /dograh)
    REGISTRY=dograhai   DOGRAH_VERSION=latest   ENABLE_TELEMETRY=true
    FORCE_TURN_RELAY=false

## Verification

    https://<host>/api/v1/health  -> 200 {"status":"ok","version":"1.51.0",...}

## Remaining step: DNS

The hostname needs a Cloudflare DNS record (proxied) pointing at the idstar.de
origin. Without it the Cloudflare edge answers 404 with an empty body, while the
origin itself serves the app correctly.
