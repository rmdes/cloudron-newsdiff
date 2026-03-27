# cloudron-newsdiff

Cloudron packaging for [NewsDiff](https://github.com/rmdes/newsdiff) — the news article diff tracker.

This repo contains only the Cloudron deployment files. The application source lives in `rmdes/newsdiff` and is included here as a git submodule.

## Contents

| File | Purpose |
|------|---------|
| `Dockerfile` | Cloudron image (cloudron/base:5.0.0) |
| `CloudronManifest.json` | Cloudron app manifest |
| `start.sh` | Container entrypoint — maps Cloudron env vars, runs migrations, starts nginx + bot + app |
| `nginx.conf` | Internal reverse proxy — routes ActivityPub paths to Botkit, everything else to SvelteKit |
| `env.sh.template` | User-editable credentials file (copied to localstorage on first run) |
| `newsdiff/` | App source (git submodule → rmdes/newsdiff) |

## Usage

### Clone with submodule

```bash
git clone --recurse-submodules https://github.com/rmdes/cloudron-newsdiff.git
cd cloudron-newsdiff
```

### Option A: Use pre-built image (quickest)

A pre-built image is published to GHCR on every push to `main`:

```bash
# Install on your Cloudron
cloudron install --image ghcr.io/rmdes/cloudron-newsdiff:main

# Update an existing install
cloudron update --app <app-id> --image ghcr.io/rmdes/cloudron-newsdiff:main
```

### Option B: Build locally

```bash
# Build the Cloudron image (pushes to your configured registry)
cloudron build

# Install on your Cloudron
cloudron install --image <your-registry>/com.newsdiff.app:<tag>

# Update an existing install
cloudron update --app <app-id> --image <your-registry>/com.newsdiff.app:<tag>
```

### Updating the app

```bash
# Pull latest app code
git submodule update --remote newsdiff
git add newsdiff
git commit -m "chore: update newsdiff to latest"

# Rebuild and redeploy
cloudron build
cloudron update --app <app-id> --image <your-registry>/com.newsdiff.app:<tag>
```

## Addons required

| Addon | Purpose |
|-------|---------|
| `postgresql` | Database |
| `redis` | Job queue (BullMQ) |
| `localstorage` | Bot profile config + uploaded images |
| `oidc` | Protects `/feeds` and `/bot/profile` routes |

## Social media credentials

On first run, `start.sh` copies `env.sh.template` to `/app/data/config/env.sh` in the app's localstorage. Edit that file via the Cloudron file manager to configure Bluesky credentials and bot identity, then restart the app.
