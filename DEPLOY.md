# Deploy (local and production)

## Overview
- Use `docker-compose.yaml` for both **development** and **production**; behavior is controlled via environment variables (see `.env.example` and `.env.prod`).

## Quick local run
1. Copy env example: `cp .env.example .env` and edit if needed.
2. Start: `docker compose up -d`
3. Add hosts entry (Windows): `127.0.0.1  traccar.local` to `C:\Windows\System32\drivers\etc\hosts`.
4. Access Traccar: http://traccar.local
5. To enable the Traefik dashboard locally, set `TRAEFIK_API_INSECURE=true` and `TRAEFIK_DASHBOARD_BIND=0.0.0.0` in your `.env` (or export the env vars) and run: `docker compose up -d`.
   For production keep `TRAEFIK_API_INSECURE=false` and `TRAEFIK_DASHBOARD_BIND=127.0.0.1` to limit dashboard access to localhost.

## Production run
1. Create `.env.prod` and set passwords and `TRACCAR_DOMAIN` (must point to your public IP/DNS). Example variables are in `.env.prod`.
2. Ensure `traefik/acme.json` exists and has secure permissions (600). On Linux: `touch traefik/acme.json && chmod 600 traefik/acme.json`.
3. Start production stack from project root:
   - `docker compose --env-file .env.prod up -d`
   (The single `docker-compose.yaml` works for both local and production; variables in `.env.prod` toggle behaviour.)
4. Verify:
   - `docker compose --env-file .env.prod ps`
   - Open `https://<your domain>`

### Enabling TLS with Let's Encrypt
- Set in `.env.prod`:
  - `TRACCAR_DOMAIN=your.public.domain` (must point to the server IP)
  - `TRACCAR_TLS=true`
  - `TRACCAR_ENTRYPOINTS=websecure`
  - `CERT_RESOLVER=le`
- Ensure ports 80 and 443 are open and routed to the host (HTTP challenge uses port 80).
- Ensure `traefik/acme.json` exists and is writable by the Docker daemon. On Linux: `touch traefik/acme.json && chmod 600 traefik/acme.json`.
- Update `traefik/traefik.yaml` and set the `email` under `certificatesResolvers.le.acme` to your contact email.
- Restart Traefik: `docker compose --env-file .env.prod up -d traefik` and watch logs: `docker compose --env-file .env.prod logs -f traefik --tail 200`.
- On success you will see ACME certificates being issued and stored in `traefik/acme.json`.

## Notes and security
- The production Traefik configuration enables ACME (Let's Encrypt). Ensure ports 80 and 443 are reachable from the Internet.
- The dashboard is disabled by default in production; enable only behind authentication.
- Store secrets (passwords) securely and **do not** commit `.env.prod` to Git.

## Troubleshooting
- Traefik logs: `docker compose logs -f traefik`
- Check routers/services in the Traefik dashboard (if enabled locally).
- If certs not issued, confirm DNS points to the host and port 80/443 are open.
