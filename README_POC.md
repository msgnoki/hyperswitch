# Hyperswitch Control Center POC

This repository snapshot is configured to spin up the Hyperswitch router and Control Center locally with Docker Compose so you can explore the dashboard with demo data. The POC profile keeps the stack lean (PostgreSQL, Redis, router, Control Center, and helper scripts) but still provisions the demo tenant so you can log in immediately.

## Prerequisites

- Docker and Docker Compose v2
- Ports `15432`, `6379`, `18080`, and `9000` available on your host
- `.oneclick-setup.env` present in the project root (already included with `ONE_CLICK_SETUP=true`)

## Start the POC stack

```bash
# from the repository root

docker compose -f docker-compose-poc.yml --env-file .oneclick-setup.env up -d
```

What this does:

- Creates Postgres (`pg`) on `localhost:15432` with `db_user/db_pass`
- Creates Redis (`redis-standalone`) on `localhost:6379`
- Boots the router (`hyperswitch-server`) on `localhost:18080`
- Boots the Control Center (`hyperswitch-control-center`) on `http://localhost:9000`
- Runs helper hooks to seed demo data and the `demo@hyperswitch.com` user

The Control Center container mounts `config/dashboard_poc.toml`, which points the UI to `http://localhost:18080`. This keeps it compatible with the custom port mapping we use here without touching the upstream `config/dashboard.toml`.

## Verify the services

```bash
# Router health check
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:18080/health
# 200 means the router is healthy
```

```bash
# List running services
 docker compose -f docker-compose-poc.yml ps
```

## Log into the Control Center

1. Open [http://localhost:9000](http://localhost:9000)
2. Sign in with `demo@hyperswitch.com` / `Hyperswitch@123`
3. If prompted for 2FA, click “Skip for now” (allowed because `force_two_factor_auth=false` in `config/docker_compose_local.toml`)
4. Explore example payments such as `poc_payment_006`, add connectors, etc.

Behind the scenes the `scripts/create_default_user.sh` container provisions the demo user and a PayPal Test connector each time the stack starts.

## Common commands

```bash
# Follow router logs
 docker compose -f docker-compose-poc.yml logs -f hyperswitch-server

# Restart just the Control Center after tweaking config/dashboard_poc.toml
 docker compose -f docker-compose-poc.yml restart hyperswitch-control-center

# Tear everything down
 docker compose -f docker-compose-poc.yml down
```

## Troubleshooting

- **Login fails immediately** – Ensure that `config/dashboard_poc.toml` still points to `http://localhost:18080`. If you switch back to the standard compose stack, remount `config/dashboard.toml` instead.
- **Ports already in use** – Stop conflicting services or change the port mappings inside `docker-compose-poc.yml` (remember to keep the Control Center config in sync).
- **Demo user missing** – Check `create-default-user` logs (`docker compose ... logs -f create-default-user`). Rerun `docker compose ... up -d create-default-user` once the router is healthy.
- **Need a clean slate** – Run `docker compose ... down -v` to drop the Postgres/Redis volumes and start over.

This README focuses on the lightweight POC workflow. For the full platform docs, refer to the upstream `README.md`.
