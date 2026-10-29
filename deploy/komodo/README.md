# OpenSEO on svc-docker

Canonical repository: https://github.com/steffzuid/open-seo
Upstream: https://github.com/every-app/open-seo

Komodo stack `open-seo` runs on server `svc-docker`, tracking `main` and
`deploy/komodo/compose.yaml`. It uses the upstream standalone stack, with an
immutable OpenSEO image digest, Cloudflare Tunnel ingress and bounded logs.
The init container exits successfully after applying migrations; this is expected.
Set Komodo stack `ignore_services` to `["init"]` so this successful one-time
container does not cause the stack to report unhealthy. It still runs on deploy.

## Access and first account

Open https://seo.zuidema.co through Cloudflare Tunnel and Cloudflare Access.
Create the administrator account on your first visit. Registration closes after
the first account. The hostname must have an Access policy before DNS is created.
Traefik routes the hostname to web on port 80; backend storage and runtime have
no host ports. A loopback-only port 3001 is retained for health checks.
`BETTER_AUTH_URL=https://seo.zuidema.co` must match the browser address.

## Configuration

Environment values live in the Komodo stack environment, never in Git.
`STORAGE_ACCESS_KEY` and a generated `STORAGE_SECRET_KEY` are set on deployment.
Keep these unchanged and back them up securely.
Set `DATAFORSEO_API_KEY` (base64 of login:password) in Komodo to enable SEO data.
Optional: `OPENROUTER_API_KEY`, Google OAuth credentials, and `CONTEXT_API_KEY`.
After changing provider settings, recreate init, web and crawl-gateway, as
explained in `docs/SELF_HOSTING_DOCKER.md`. Do not just restart containers.

## Persistence, backup and updates

Docker named volumes `open-seo_storage`, `open-seo_runtime`, `open-seo_config`
and `open-seo_web-config` persist across redeploys. Never remove them or use
`docker compose down -v`. Back up both state and configuration; the config
volume contains generated credentials. Guest-level snapshot coverage should be
verified separately; no additional scheduled backup is installed by this stack.

Use the upstream `scripts/celld/backup.mjs` instructions, passing the Komodo
compose file and the same project name/environment, before updating. Store
backup archives privately. Restore only to an empty installation with the
matching image and storage credentials.

Updates are intentional: sync upstream, review changes, choose a new image
digest in this compose file, back up, then deploy the stack in Komodo. Automatic
image updates are disabled. Migrations run during init; rollback requires the
matching pre-update backup rather than only switching the image.

Check `curl --fail http://127.0.0.1:3001/api/health` on svc-docker and inspect
init/runtime logs in Komodo after deployment.
