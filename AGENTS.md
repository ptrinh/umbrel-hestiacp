# AGENTS.md — umbrel-hestiacp

Guidance for AI coding agents working in this repo.

## What this repo is
An Umbrel **Community App Store** that ships one app: **HestiaCP** (Hestia
Control Panel). Users add the repo URL in Umbrel → App Store → Community App
Stores, then install "Hestia Control Panel".

## Layout
- `umbrel-app-store.yml` — store identity (`id: trinh`). App ids must be
  prefixed with the store id, e.g. `trinh-hestiacp`.
- `trinh-hestiacp/umbrel-app.yml` — app manifest (HestiaCP 1.9.6, port 8083,
  `deterministicPassword`, `defaultUsername: admin`).
- `trinh-hestiacp/docker-compose.yml` — three services: `hestia`
  (`smied/hestia-cp`, privileged), `sslproxy` (`alpine/socat` HTTP→TLS tunnel),
  and the Umbrel `app_proxy`.
- `trinh-hestiacp/hooks/post-start` — sets HestiaCP `admin` password to
  `$APP_PASSWORD` once (sentinel-guarded). Calls HestiaCP CLI by **absolute
  path** (`/usr/local/hestia/bin/...`) because `docker exec` is non-login.

## Critical invariants (don't break)
- The app reaches the panel via `app_proxy → sslproxy(socat) → hestia:8083`.
  `app_proxy` must point at the **socat** service over **HTTP**, not at HestiaCP
  directly (HestiaCP serves self-signed HTTPS → app_proxy 502s).
- Container DNS names are `trinh-hestiacp_<service>_1`; `APP_HOST` and the socat
  `OPENSSL-CONNECT` target depend on these exact names.
- Pin images by digest. Keep YAML valid (`ruby -ryaml -e 'YAML.load_file(...)'`).

## Testing
Install on a real Umbrel via `umbreld client appStore.addRepository.mutate
--url <repo>` then `apps.install.mutate --appId trinh-hestiacp`. Verify the host
app port returns the HestiaCP login (302 → LOGIN page), not a 502.

## Related repo
Non-privileged, multi-arch image (official-store path):
https://github.com/ptrinh/hestiacp-docker
