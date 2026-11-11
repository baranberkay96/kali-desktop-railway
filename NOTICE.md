# NOTICE: Kali Linux KDE Desktop in Your Browser template

## Base image and its components

The service is built `FROM ghcr.io/linuxserver/baseimage-selkies:kali@sha256:bcccb233834596ffc8db2c9c8c2bcc23b2ba77d81792288dff7360c74d318297` (the `kali` tag on 2026-11-08), the base that linuxserver.io's own `kali-linux` image is built on. The Dockerfile repeats the steps of linuxserver.io's `kali-linux` Dockerfile (git revision `11d330ff53bebdbe0fd43442ddcb1f5d45b78c01`) with a smaller package list: their finished image is 4.75 GB compressed, above the 4 GB image size limit of Railway's Trial and Free plans.

| Component | Source | License | How it is used |
|---|---|---|---|
| linuxserver.io baseimage-selkies (`kali`) | https://github.com/linuxserver/docker-baseimage-selkies | GPL-3.0 | Base image (`FROM`): the s6 service definitions, `init-nginx`, `/defaults/default.conf`, Selkies |
| linuxserver.io `docker-kali-linux`, files under `root/` | https://github.com/linuxserver/docker-kali-linux at `11d330ff` | GPL-3.0 | Copied unmodified to `services/desktop/root/`: `/defaults/startwm_wayland.sh` (KDE Wayland session start), `/defaults/startwm.sh`, `/defaults/autostart`, `/etc/polkit-1/rules.d/49-lsio-sudo.rules` |
| linuxserver.io `selkies-layers:amd64-kali-kwin` | https://github.com/linuxserver/docker-baseimage-selkies (published layer), pinned by digest `sha256:dac93cfd…` | GPL-2.0-or-later (KWin) | KWin library with Selkies screen capture, copied in as upstream does (`COPY --from`) |
| Kali icon for the Selkies page | https://github.com/linuxserver/docker-templates at `bfd0e385`, `linuxserver.io/img/kali-logo.png` | GPL-3.0 (repository); the Kali logo is a trademark of OffSec Services Limited | `services/desktop/kali-icon.png`, unmodified |
| wl-clipboard-rs-tools 0.9.5 (`wl-copy`, `wl-paste`) | https://crates.io/crates/wl-clipboard-rs-tools | MIT OR Apache-2.0 | Compiled from crates.io in a build stage (`rust:1.99.1-slim-trixie`), as upstream does |
| Selkies | https://github.com/selkies-project/selkies | MPL-2.0 | Streaming server and web client, in the base image |
| s6-overlay 3.2.1.0 | https://github.com/just-containers/s6-overlay, https://skarnet.org/software/ | ISC | Process supervision. `railway-start` calls `s6-svscan`, `s6-rc-compile`, `s6-rc-init` and `s6-rc` from the image |
| catatonit | https://github.com/openSUSE/catatonit (distribution package in the base image) | GPL-2.0-or-later | Container init and subreaper (`ENTRYPOINT`) |
| Kali Linux (kali-rolling) packages: KDE Plasma (`kali-desktop-kde` and the KDE apps upstream installs), `kali-linux-core`, `kali-tools-top10`, Firefox ESR, dirb, ffuf, gobuster, whatweb, exploitdb, wordlists and others | https://www.kali.org/ and https://pkg.kali.org/ | Each package's own license (`/usr/share/doc/*/copyright` in the image) | Installed with apt at build time, unmodified |

### GPL notes

- The GPL parts stay under their licenses. Their complete corresponding source is published by their authors at the repositories above, at the git revisions named here.
- **Copied files:** the four files under `services/desktop/root/` and `kali-icon.png` are unmodified copies from linuxserver.io (GPL-3.0); their license applies to them.
- **Modified files:** at build time, `services/desktop/patch-image.sh` edits `/defaults/default.conf` (GPL-3.0, from docker-baseimage-selkies: basic auth switched on, an Origin check, anti-framing headers and a `/healthz` location) and `/etc/nginx/nginx.conf` (2 worker processes, error log on stderr). The script and the inserted snippets (`nginx-origin-map.conf`, `nginx-server-hardening.conf`) are the complete record of these changes. Anyone who distributes a built image of this template should keep them available with it.
- The files written for this template (`Dockerfile`, `railway-start.sh`, `pkg-restore.sh`, `patch-image.sh`, the two nginx snippets and the documentation) are offered under GPL-3.0-or-later.
- `railway-start` re-implements, in its own code, the order of steps that s6-overlay's `preinit`, `stage0` and `rc.init` perform, and then calls the image's own s6 tools and service definitions.

## Market templates reviewed

`kali-linux-web-desktop` "Kali Linux (Web Desktop)" and the Kali web-terminal templates (config read only): read for ideas only. Nothing copied.

Product names belong to their owners. This is a community template, not affiliated with linuxserver.io, the Selkies project, the distribution's publishers or Railway. Kali Linux is a trademark of OffSec Services Limited. KDE® and Plasma are trademarks of KDE e.V.
