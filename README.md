# Hestia Control Panel — Umbrel Community App Store

Run [HestiaCP](https://hestiacp.com), a full open-source web-hosting control
panel (websites, databases, DNS, mail, FTP), as an app on your
[Umbrel](https://umbrel.com).

This repository is an Umbrel **Community App Store** — add its URL in Umbrel and
HestiaCP shows up in your App Store.

---

## Install

1. Open your Umbrel dashboard → **App Store**.
2. Top-right **⋮** menu → **Community App Stores**.
3. Paste this repo's URL:
   ```
   https://github.com/ptrinh/umbrel-hestiacp
   ```
   and click **Add**.
4. Open the **"Trinh App Store"** section → install **Hestia Control Panel**.
5. First start takes a few minutes while HestiaCP initialises. Then click
   **Open**.

**Login:** username `admin`, password = the app password shown on the app's
page in Umbrel (set automatically on first start — see below).

> The panel uses a **self-signed TLS certificate**, so your browser shows a
> one-time certificate warning. That's expected — proceed past it.

---

## How it works

```
browser ──HTTP──▶ Umbrel app_proxy ──HTTP──▶ socat sidecar ──TLS──▶ HestiaCP :8083
```

HestiaCP only serves its panel over **HTTPS with a self-signed certificate** on
port 8083. Umbrel's `app_proxy` validates upstream TLS and has no option to skip
it, so it can't talk to HestiaCP directly (you'd get a `502`). To bridge that,
the app adds a tiny [`socat`](https://hub.docker.com/r/alpine/socat) sidecar:
`app_proxy` speaks plain HTTP to the sidecar, and the sidecar transparently
wraps the byte-stream in TLS to HestiaCP — so HestiaCP still sees a real HTTPS
request (no scheme/redirect/Host rewriting issues), and the cert is ignored.

The `hestia` service runs the upstream
[`smied/hestia-cp`](https://hub.docker.com/r/smied/hestia-cp) image (HestiaCP
1.9.6), pinned by digest. It is **privileged** (systemd-based; HestiaCP manages
system services and would need the firewall) — which is why this ships as a
*community* app, not the official store (see below).

### Credentials

The app declares `deterministicPassword`, so Umbrel derives a per-device app
password and shows it on the app page. A `post-start` hook sets HestiaCP's
`admin` password to that value on first boot (once — guarded by a sentinel
file, so it never overwrites a password you later change in the panel).

---

## Containers

| Service | Image | Role |
|---|---|---|
| `hestia` | `smied/hestia-cp` (pinned) | HestiaCP itself (panel on 8083) |
| `sslproxy` | `alpine/socat` (pinned) | HTTP→TLS tunnel so app_proxy can reach the panel |
| `app_proxy` | Umbrel built-in | Umbrel's reverse proxy / Open button |

The web UI (8083) is reached through `app_proxy`. Mail/DNS/FTP ports are **not**
published to the host by default to avoid clashing with Umbrel's own services —
add a `ports:` mapping in `trinh-hestiacp/docker-compose.yml` if you need them
(pick host ports that don't collide).

## Data & backups

HestiaCP state lives in Docker **named volumes** (`hestia_home`, `hestia_usr`,
`hestia_etc`, `hestia_var`, `hestia_backup`) because the image populates `/usr`,
`/etc` and `/var` from itself on first run (bind-mounting empty dirs would
shadow them). These named volumes are **not** covered by Umbrel's built-in app
backup — use HestiaCP's own backup tools for account data.

## Status

✅ Verified end-to-end on a real Umbrel: install → Open button reaches the
HestiaCP login → admin password auto-set.

---

## Why not the official Umbrel App Store?

The official store requires **non-privileged**, **multi-architecture**,
digest-pinned images. The image used here is privileged (systemd) and
`linux/amd64`-only. A non-privileged, multi-arch HestiaCP image is being built
separately at **[ptrinh/hestiacp-docker](https://github.com/ptrinh/hestiacp-docker)**
as the path toward an official-store submission. Once that image is proven, this
app can switch to it and be submitted upstream.

## License

Packaging files are provided as-is. HestiaCP is © the HestiaCP project (GP-3.0).
