# CLAUDE.md — cloudron-newsdiff

Cloudron packaging for [NewsDiff](https://github.com/rmdes/newsdiff).
The app source lives in the `newsdiff/` git submodule. This repo only contains deployment files.

## Repository Layout

```
cloudron-newsdiff/
├── newsdiff/            # git submodule — rmdes/newsdiff (app source)
├── Dockerfile           # cloudron/base:5.0.0, builds from newsdiff/ submodule
├── CloudronManifest.json
├── start.sh             # entrypoint: maps Cloudron env vars, runs migrations, starts processes
├── nginx.conf           # reverse proxy: ActivityPub paths → Botkit, everything else → SvelteKit
└── env.sh.template      # user-editable credentials (copied to localstorage on first run)
```

## Commands

### First-time install

```bash
# 1. Clone with submodule
git clone --recurse-submodules https://github.com/rmdes/cloudron-newsdiff.git
cd cloudron-newsdiff

# 2. Build the Docker image (pushes to your configured registry)
cloudron build

# 3. Install on Cloudron
cloudron install --image <registry>/<image>:<tag> --location <subdomain.yourdomain.com>
```

### Update an existing install

```bash
# 1. Build new image from current submodule
cloudron build

# 2. Find the app ID if you don't have it
cloudron list

# 3. Update the running app
cloudron update --app <app-id> --image <registry>/<image>:<tag>
```

### Update the app code (pull new commits from rmdes/newsdiff)

```bash
# 1. Pull latest commits from the app repo into the submodule
git submodule update --remote newsdiff

# 2. Commit the pinned submodule bump
git add newsdiff
git commit -m "chore: update newsdiff to latest"
git push

# 3. Rebuild and redeploy
cloudron build
cloudron update --app <app-id> --image <registry>/<image>:<tag>
```

### Check current submodule version

```bash
git submodule status
# shows the pinned commit SHA and whether it's ahead/behind
```

## How start.sh works

On every container start, `start.sh`:
1. Creates required directories under `/app/data/` and `/run/`
2. Copies `env.sh.template` to `/app/data/config/env.sh` on first run (localstorage)
3. Sources `/app/data/config/env.sh` for user-supplied credentials (Bluesky, bot identity)
4. Maps Cloudron addon env vars (`CLOUDRON_POSTGRESQL_*`, `CLOUDRON_REDIS_*`, etc.) to app vars
5. Runs DB migrations via `node --import tsx/esm src/lib/server/db/migrate.ts`
6. Starts nginx (port 8000, public), Botkit (port 8001), and SvelteKit (port 3000)
7. Uses `wait -n` — if any process exits, the container stops

## Cloudron Addons

| Addon | Env vars injected | Used for |
|-------|------------------|---------|
| `postgresql` | `CLOUDRON_POSTGRESQL_*` | Database |
| `redis` | `CLOUDRON_REDIS_*` | BullMQ job queue |
| `localstorage` | `CLOUDRON_DATA_DIR` | Bot profile config + uploaded images |
| `oidc` | `CLOUDRON_OIDC_*` | Protects `/feeds` and `/bot/profile` |

## Configuring Social Credentials

After install, edit `/app/data/config/env.sh` via the Cloudron file manager:

```bash
export BLUESKY_HANDLE="yourhandle.bsky.social"
export BLUESKY_PASSWORD="xxxx-xxxx-xxxx-xxxx"
export BOT_USERNAME="bot"
export BOT_NAME="NewsDiff Bot"
```

Restart the app for changes to take effect.
