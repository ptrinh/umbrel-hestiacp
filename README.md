# Trinh — Umbrel Community App Store

A community app store for [Umbrel](https://umbrel.com).

## Add this store to your Umbrel

1. Open your Umbrel dashboard → **App Store**.
2. Click the **⋮** menu (top right) → **Community App Stores**.
3. Paste this repository's URL and click **Add**.
4. The apps below appear under the **"Trinh App Store"** section.

## Apps

### Hestia Control Panel (`trinh-hestiacp`)

[HestiaCP](https://hestiacp.com) — a full open-source web hosting control panel
(websites, email, databases, DNS, FTP) packaged for Umbrel.

**Login:** username `admin`, password = the app password shown on the app's
page in Umbrel (it's set automatically on first start).

**First-time notes**

- The panel uses a **self-signed TLS certificate**, so your browser shows a
  certificate warning the first time — accept it to continue.
- First boot takes a few minutes while HestiaCP initialises all its services.
- This is a **privileged, systemd-based** container (HestiaCP manages system
  services and the firewall). It is heavier than a typical Umbrel app.

**What is / isn't exposed**

- The web UI (port 8083) is reached through Umbrel's app proxy — that's the
  **Open** button.
- Mail/DNS/FTP ports are not published to the host by default to avoid clashing
  with Umbrel's own services. Add a `ports:` mapping in
  `trinh-hestiacp/docker-compose.yml` if you need them (pick host ports that
  don't collide with Umbrel or other apps).

**Why this isn't in the official Umbrel App Store (yet)**

The official store requires non-privileged, multi-architecture, pinned images.
HestiaCP currently needs a privileged systemd container and the only usable
public image (`smied/hestia-cp`) is `linux/amd64` only. A non-privileged,
multi-arch image is being explored separately as the path to upstream
submission.

## Data & backups

HestiaCP state lives in Docker named volumes (`hestia_home`, `hestia_usr`,
`hestia_etc`, `hestia_var`, `hestia_backup`) because the image populates
`/usr`, `/etc` and `/var` from itself on first run. These volumes are **not**
covered by Umbrel's built-in app backup — use HestiaCP's own backup tools for
account data.
