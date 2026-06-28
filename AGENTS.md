# AGENTS.md - umbrel-hestiacp

Guidance for AI coding agents working in this repo.

## What this repo is
An Umbrel **Community App Store** that ships one app: **HestiaCP** (Hestia
Control Panel), as a non-privileged, multi-arch build. Users add the repo URL
in Umbrel -> App Store -> Community App Stores, then install "Hestia Control
Panel".

## Layout
- `umbrel-app-store.yml` - store identity (`id: trinh`). App ids must be
  prefixed with the store id, e.g. `trinh-hestiacp-np`.
- `trinh-hestiacp-np/umbrel-app.yml` - app manifest (HestiaCP 1.9.6, port 8084,
  `deterministicPassword`, `defaultUsername: admin`).
- `trinh-hestiacp-np/docker-compose.yml` - three services: `hestia`
  (`ghcr.io/ptrinh/hestiacp`, non-privileged, multi-arch, pinned by digest),
  `sslproxy` (`alpine/socat` HTTP->TLS tunnel), and the Umbrel `app_proxy`.
  The admin password is set to `$APP_PASSWORD` on first start by the image
  entrypoint (sentinel-guarded), so no post-start hook is needed.

## Critical invariants (don't break)
- The app reaches the panel via `app_proxy -> sslproxy(socat) -> hestia:8083`.
  `app_proxy` must point at the **socat** service over **HTTP**, not at HestiaCP
  directly (HestiaCP serves self-signed HTTPS -> app_proxy 502s).
- Container DNS names are `trinh-hestiacp-np_<service>_1`; `APP_HOST` and the
  socat `OPENSSL-CONNECT` target depend on these exact names.
- Pin the image by digest (`ghcr.io/ptrinh/hestiacp:build-<run_id>@sha256:...`).
  Keep YAML valid (`ruby -ryaml -e 'YAML.load_file(...)'`).
- Persist the data volumes: `home`, `mysql`, `postgresql`, `hestia`,
  `hestia-conf`, `backup` under `${APP_DATA_DIR}/data/`.

## Testing
Install on a real Umbrel via `umbreld client appStore.addRepository.mutate
--url <repo>` then `apps.install.mutate --appId trinh-hestiacp-np`. Verify the
app port returns the HestiaCP login page (a request renders the login form,
not a 502), then log in with `admin` + the app password.

## Related repo
The image source (Dockerfile, CI, docs) - also the official-store submission:
https://github.com/ptrinh/hestiacp-docker
