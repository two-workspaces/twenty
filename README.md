# Deploy and Host Twenty CRM on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/twenty-crm-production?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=twenty)

[Twenty](https://twenty.com/) is the open-source CRM: pipelines, contacts, companies, notes, tasks, email sync, and workflow automation in a fast modern UI — a self-hosted alternative to Salesforce and Pipedrive, built by one of the most active open-source teams (50K+ GitHub stars).

## About Hosting Twenty CRM

This template runs Twenty v2 as four Railway services: the server (API + web app, runs migrations and cron registration on boot), a dedicated BullMQ worker (emails, imports, workflow runs — kept off the request path), PostgreSQL 16, and Redis tuned with `noeviction` as upstream requires for its queues. File attachments go to a **Railway bucket over S3** — Twenty's storage driver checks the specific bucket with `headBucket`, which works cleanly with Railway's bucket-scoped credentials, and it also solves the shared-volume problem (upstream's compose shares a local-storage volume between server and worker, which Railway services can't do). The Redis URL carries `?family=0` for Railway's IPv6-only private networking. First boot migrates the database in a minute or two; then create your workspace at the server's domain.

## Common Use Cases

- Sales CRM: pipelines, deals, contacts and companies with a spreadsheet-fast UI
- Replacing Salesforce/Pipedrive seats with a self-hosted CRM your data stays on
- Pair with a Chatwoot deployment for the full self-hosted customer stack: CRM + support desk

## Dependencies for Twenty CRM Hosting

- All bundled: PostgreSQL 16, Redis 7, and a Railway bucket for file attachments
- SMTP credentials (optional, post-deploy) for email notifications; Google/Microsoft OAuth apps (optional) for email and calendar sync

### Deployment Dependencies

- [Twenty self-hosting documentation](https://twenty.com/developers/section/self-hosting)
- [Template source on GitHub](https://github.com/two-workspaces/twenty)

### Implementation Details

**Your Twenty URL is the `server` service's domain.** First visit creates your workspace and admin user.

Service map:

| Service | Image | Role |
|---|---|---|
| server | `twentycrm/twenty:v2.51.0` | API + web app, public domain, healthcheck `/healthz`, runs migrations |
| worker | `twentycrm/twenty:v2.51.0` (CMD `yarn worker:prod`, after the server finishes upgrading the database) | BullMQ jobs: emails, imports, workflows |
| postgres | `postgres:16-alpine` | Database (`default`), on a volume |
| redis | `redis:7-alpine` | BullMQ queues (`noeviction`, append-only, on a volume) |

Variable wiring (lives in the template composer; documented here for maintenance):

- **Shared by server / worker**: `PG_DATABASE_URL=postgres://postgres:${{postgres.POSTGRES_PASSWORD}}@${{postgres.RAILWAY_PRIVATE_DOMAIN}}:5432/default`, `REDIS_URL=redis://:${{redis.REDIS_PASSWORD}}@${{redis.RAILWAY_PRIVATE_DOMAIN}}:6379?family=0` (the `family=0` suffix is required — ioredis defaults to IPv4 and Railway private DNS is IPv6-only), `SERVER_URL=https://${{server.RAILWAY_PUBLIC_DOMAIN}}`, `APP_SECRET` (generated on server, referenced by worker — must match), `STORAGE_S3_NAME=${{twenty-storage.BUCKET}}`, `STORAGE_S3_ENDPOINT=${{twenty-storage.ENDPOINT}}`, `STORAGE_S3_ACCESS_KEY_ID=${{twenty-storage.ACCESS_KEY_ID}}`, `STORAGE_S3_SECRET_ACCESS_KEY=${{twenty-storage.SECRET_ACCESS_KEY}}`
- **server**: `PORT=3000` (explicit service variable — Railway injects `PORT=8080` over image ENV)
- **postgres**: `POSTGRES_PASSWORD` (generated); volume at `/var/lib/postgresql/data`
- **redis**: `REDIS_PASSWORD` (generated); volume at `/data`

Notes and limits:

- Email/calendar sync (Gmail, Microsoft) needs your own OAuth app credentials — add the `MESSAGING_PROVIDER_*` / `AUTH_GOOGLE_*` / `AUTH_MICROSOFT_*` variables per Twenty's docs.
- Without SMTP variables, invitation and notification emails are skipped; the app works fully otherwise.
- Scale the `worker` replicas if imports or workflow executions queue up.
- Upgrade by bumping the image tag in both `server` and `worker` — migrations run automatically on the server, and the worker waits for them before it starts.
- Upgrading a deploy made from the earlier v2.20 version of this template: records carry over, and everyone is signed out once and signs in again.

## Why Deploy Twenty CRM on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Twenty CRM on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
