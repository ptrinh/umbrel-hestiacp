# Hestia Control Panel - Umbrel Community App Store

Run [HestiaCP](https://hestiacp.com), a full open-source web-hosting control
panel (websites, databases, DNS, mail, FTP), as an app on your
[Umbrel](https://umbrel.com).

This repository is an Umbrel **Community App Store** - add its URL in Umbrel and
HestiaCP shows up in your App Store.

---

## Install

1. Open your Umbrel dashboard -> **App Store**.
2. Top-right menu -> **Community App Stores**.
3. Paste this repo's URL:
   ```
   https://github.com/ptrinh/umbrel-hestiacp
   ```
   and click **Add**.
4. Open the **"Trinh"** store section and install **Hestia Control Panel**.
5. First start takes a few minutes while HestiaCP initialises. Then click
   **Open**.

**Login:** username `admin`, password = the app password shown on the app's
page in Umbrel (set automatically on first start).

> The panel uses a **self-signed TLS certificate**, so your browser shows a
> one-time certificate warning. That's expected - proceed past it.

---

## What you get

A **non-privileged, multi-arch (amd64 + arm64)** HestiaCP build. The standard
HestiaCP container runs systemd as PID 1 and needs `privileged: true`; this app
uses a purpose-built image
([ptrinh/hestiacp-docker](https://github.com/ptrinh/hestiacp-docker)) that
replaces systemd with a `systemctl` shim and disables the privilege-requiring
features (firewall / fail2ban / disk quota), so it runs with **no privileged
mode and no extra Linux capabilities**.

The full panel is intact and reachable through Umbrel's single app port:
nginx + Apache, PHP-FPM, MariaDB + PostgreSQL, mail (Exim / Dovecot /
Roundcube), DNS (Bind), FTP (vsftpd), File Manager, cron, phpMyAdmin and
phpPgAdmin.

Trade-off: **no in-panel firewall/fail2ban** - handle network security at the
Umbrel/router level.

---

## How it works

```
browser --HTTP--> Umbrel app_proxy --HTTP--> socat sidecar --TLS--> HestiaCP :8083
```

HestiaCP only serves its panel over **HTTPS with a self-signed certificate** on
port 8083. Umbrel's `app_proxy` validates upstream TLS and has no option to skip
it, so it can't talk to HestiaCP directly (you'd get a `502`). To bridge that,
the app adds a tiny [`socat`](https://hub.docker.com/r/alpine/socat) sidecar:
`app_proxy` speaks plain HTTP to the sidecar, and the sidecar wraps the
byte-stream in TLS to HestiaCP - so HestiaCP still sees a real HTTPS request and
the cert is ignored.

### Credentials
The app declares `deterministicPassword`, so Umbrel derives a per-device app
password and shows it on the app page. The image entrypoint sets HestiaCP's
`admin` password to that value on first boot (once - guarded by a sentinel file,
so it never overwrites a password you later change in the panel).

---

## Data and backups

Data persists under the app's `${APP_DATA_DIR}/data/*` (MariaDB, PostgreSQL,
HestiaCP config, `/home`, backups); the image seeds these on first run and
survives container recreates (including the container IP changing). Use
HestiaCP's own backup tools for account data; Umbrel's built-in app backup does
not cover these volumes.

The app does not publish Umbrel's ports 80/443. Hosted sites are published on
alternate ports (8088/8448); add or change `ports:` mappings in the app's
`docker-compose.yml` if you need others.

---

## Status

Verified end-to-end on a real Umbrel: install, **Open** reaches the HestiaCP
login, admin password auto-set, create user, web hosting (domains serve),
databases (MariaDB + PostgreSQL), mail, and phpMyAdmin/phpPgAdmin. Confirmed to
run `Privileged=false`, and to keep working after a container recreate with a
changed IP.

## Official Umbrel App Store

This same non-privileged, multi-arch image is the basis of a submission to the
**official** Umbrel App Store:
[getumbrel/umbrel-apps#5773](https://github.com/getumbrel/umbrel-apps/pull/5773).
Image + Dockerfile + CI: [ptrinh/hestiacp-docker](https://github.com/ptrinh/hestiacp-docker).

## License

Packaging files are provided as-is. HestiaCP is (c) the HestiaCP project (GPL-3.0).
