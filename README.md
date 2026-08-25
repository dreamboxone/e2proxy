# e2proxy

Proxy-bridge plugin for **Enigma2** (works on **DreamOS** and OpenEmbedded
images such as OpenATV / OpenPLi). It routes the receiver's internet traffic
through a phone-hosted proxy (e.g. the Android app **EveryProxy**), so the box
uses the phone's unfiltered internet. Live DVB TV is unaffected (it is not IP
traffic), matching the intended use.

Telegram: **https://t.me/routekernel1**

## Features

- On/off control from the plugin screen and from the Extensions menu.
- Plugin version is shown in the plugin menu and on the main screen.
- Three status LEDs: **Receiver network**, **Internet access**, **Proxy status**.
- Two routing modes, selectable in Settings:
  - **Transparent** – all box TCP is redirected through the proxy with
    `iptables REDIRECT` + a small built-in Python redirector (a redsocks
    equivalent using PySocks and `SO_ORIGINAL_DST`). DNS is tunnelled too.
    If `iptables`/nat is unavailable it automatically falls back to *simple*.
  - **Simple** – sets proxy for updates / proxy-aware tools (connman, apt,
    opkg, `http_proxy`).
- Proxy types: **HTTP, HTTPS, SOCKS4, SOCKS5**. Username/Password are enabled
  for the types that support authentication (SOCKS4 has none).
- Diagnostic log at `/tmp/e2proxy.log` (tmpfs → cleared on every reboot),
  hard-capped at **512 KiB**. Records receiver name, model, image,
  architecture, CPU, plugin version and connect/disconnect events – for
  troubleshooting only.
- Full restore on OFF: the dedicated `E2PROXY` iptables chain, DNS rule and
  all simple-mode files are removed and the redirector is stopped.
- Self-contained: **PySocks is vendored** (`_socks.py`), no external Python
  dependency required.
- **Integrity protection**: every shipped file is listed in `manifest.json`
  and the manifest is signed with the author's private RSA key. At runtime the
  plugin verifies the signature and each file hash with a dependency-free,
  pure-Python RSA verifier; tampering is detected and logged, and starting the
  proxy is refused. Copyright/licence header on every source file.

## Install

Copy the package to the receiver's `/tmp` and install:

```bash
# DreamOS
dpkg -i /tmp/e2proxy_<version>_all.deb
# OE images
opkg install /tmp/e2proxy_<version>_all.ipk
```

After install the plugin prints an English notice and **restarts Enigma2**.
The installer file is removed from `/tmp` automatically by the post-install
script.

## Notes

- The package is `Architecture: all` (pure Python, no compiled binaries), so a
  single build runs on every CPU architecture.
- `HTTPS` proxy is handled as HTTP CONNECT (PySocks does not do TLS-to-proxy).
- Transparent DNS tunnelling works best with SOCKS5; for HTTP/HTTPS proxies it
  is best-effort and can be turned off in Settings.

## Third-party

Bundles **PySocks** (`_socks.py`, BSD-3-Clause, © 2006 Dan-Haim). See the file
header. All other files are proprietary – see `LICENSE`.
