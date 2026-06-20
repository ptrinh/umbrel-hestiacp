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
4. Open the **"Trinh App Store"** section and install one of the two apps below.
5. First start takes a few minutes while HestiaCP initialises. Then click
   **Open**.

**Login:** username `admin`, password = the app password shown on the app's
page in Umbrel (set automatically on first start).

> The panel uses a **self-signed TLS certificate**, so your browser shows a
> one-time certificate warning. That's expected — proceed past it.

---

## Two variants

This store ships two HestiaCP apps. **The non-privileged build is recommended.**

| App | Image | Privileged? | Arch | Best for |
|---|---|---|---|---|
| **Hestia Control Panel (non-privileged)** — `trinh-hestiacp-np` | `ghcr.io/ptrinh/hestiacp` (custom) | **No** — no extra caps | amd64 **+ arm64** | Almost everyone, incl. Raspberry-Pi-class boards. Same build as the official-store submission. |
| Hestia Control Panel — `trinh-hestiacp` | `smied/hestia-cp` | **Yes** (systemd) | amd64 only | Folks who specifically want the full systemd image (incl. in-panel firewall/fail2ban). |

### Why a non-privileged build exists
The standard HestiaCP container runs **systemd as PID 1** and needs
`privileged: true`. The non-privileged variant uses a purpose-built image
([ptrinh/hestiacp-docker](https://github.com/ptrinh/hestiacp-docker)) that
replaces systemd with a `systemctl` shim and disables the privilege-requiring
features (firewall / fail2ban / disk quota), so it runs with **no privileged
mode and no extra Linux capabilities**. The web/DB stack (Nginx + Apache +
PHP-FPM + MariaDB) is fully intact.

Trade-off: the non-privileged build has **no in-panel firewall/fail2ban** by
default — handle network security at the Umbrel/router level. (The privileged
variant keeps them.)

---

## How it works (both variants)

```
browser ──HTTP──▶ Umbrel app_proxy ──HTTP──▶ socat sidecar ──TLS──▶ HestiaCP :8083
```

HestiaCP only serves its panel over **HTTPS with a self-signed certificate** on
port 8083. Umbrel's `app_proxy` validates upstream TLS and has no option to skip
it, so it can't talk to HestiaCP directly (you'd get a `502`). To bridge that,
each app adds a tiny [`socat`](https://hub.docker.com/r/alpine/socat) sidecar:
`app_proxy` speaks plain HTTP to the sidecar, and the sidecar transparently
wraps the byte-stream in TLS to HestiaCP — so HestiaCP still sees a real HTTPS
request, and the cert is ignored.

### Credentials
Both apps declare `deterministicPassword`, so Umbrel derives a per-device app
password and shows it on the app page. A `post-start` hook sets HestiaCP's
`admin` password to that value on first boot (once — guarded by a sentinel file,
so it never overwrites a password you later change in the panel).

---

## Data & backups

- **Non-privileged (`trinh-hestiacp-np`)** — data persists under the app's
  `${APP_DATA_DIR}/data/*` (MariaDB, HestiaCP config, `/home`, backups); the
  image seeds these on first run.
- **Privileged (`trinh-hestiacp`)** — state lives in Docker **named volumes**
  (the `smied` image populates `/usr`, `/etc`, `/var` from itself, so empty bind
  mounts would shadow them).

Either way, use HestiaCP's own backup tools for account data; Umbrel's built-in
app backup does not cover these volumes.

By default neither app publishes Umbrel's ports 80/443. The non-privileged app
publishes hosted-site HTTP/HTTPS on alternate ports (9088/9448); add or change
`ports:` mappings in the app's `docker-compose.yml` if you need others.

---

## Status

✅ Both variants verified end-to-end on a real Umbrel: install → **Open** button
reaches the HestiaCP login → admin password auto-set. The non-privileged variant
additionally confirmed to run `Privileged=false` with `${APP_DATA_DIR}`
persistence across container recreation.

## Official Umbrel App Store

The non-privileged, multi-arch image is the basis of a submission to the
**official** Umbrel App Store:
[getumbrel/umbrel-apps#5773](https://github.com/getumbrel/umbrel-apps/pull/5773).
Image + Dockerfile + CI: [ptrinh/hestiacp-docker](https://github.com/ptrinh/hestiacp-docker).

## License

Packaging files are provided as-is. HestiaCP is © the HestiaCP project (GPL-3.0).
