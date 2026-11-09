# syntax=docker/dockerfile:1
#
# Kali Linux 2026.3 (kali-rolling) with KDE Plasma (Wayland), in the browser, for Railway.
#
# Built the way linuxserver.io builds its kali-linux image (Dockerfile and
# root/ of linuxserver/docker-kali-linux, git revision 11d330ff), on the same
# base (baseimage-selkies, kali), but with a smaller tool set: the full image
# is 4.75 GB compressed, above the 4 GB image size limit of Railway's Trial
# and Free plans, so every deploy on those plans failed. This one stays well
# below it. Add more tools with APT_PACKAGES (see the README).
#
# The base is pinned by digest (multi-arch index of the "kali" tag, 2026-11-08).
# To upgrade, replace the digest with a newer one and redeploy.

# wl-copy/wl-paste written in Rust, as in the linuxserver.io image: they use
# KWin's data-control protocol, so the clipboard works without a focus window.
FROM rust:1.99.1-slim-trixie@sha256:f6c4a35d21dbb3c89d7a8d056c193e94fc09037f2b58dd858b09c39a47dd02d7 AS clipboard
RUN cargo install --locked --root /out wl-clipboard-rs-tools@0.9.5

FROM ghcr.io/linuxserver/baseimage-selkies:kali@sha256:bcccb233834596ffc8db2c9c8c2bcc23b2ba77d81792288dff7360c74d318297

# Image defaults for Railway. Variables set on the Railway service win.
#  - PUID/PGID 1000: fixed owner of the /config volume. Do not change them
#    after the first boot, because files already on the volume keep the old ids.
#  - START_DOCKER=false: Railway gives no privileged containers, so the
#    bundled Docker daemon can never run there.
#  - SELKIES_ALLOWED_ORIGINS empty: the image ships "*", which lets any
#    website open the desktop's WebSocket with the browser's cached login.
#    Empty is Selkies' own default: same-origin only.
#  - SELKIES_ENABLE_BASIC_AUTH=false: nginx does the login. Selkies listens
#    on loopback only.
#  - PIXELFLUX_WAYLAND=true: KDE runs as a Wayland session, as upstream.
ENV PUID=1000 \
    PGID=1000 \
    START_DOCKER=false \
    SELKIES_ALLOWED_ORIGINS="" \
    SELKIES_ENABLE_BASIC_AUTH=false \
    PIXELFLUX_WAYLAND=true \
    TITLE="Kali Linux 2026.3 KDE Desktop" \
    CUSTOM_USER=kali \
    PORT=3000

# KDE Plasma as in the linuxserver.io image, plus Kali's top 10 tools
# (kali-tools-top10: Metasploit, Burp Suite, nmap, sqlmap, Hydra, John,
# Wireshark, Aircrack-ng, NetExec, Responder), Firefox and a few common
# command-line tools. The large kali-linux-default set is left out to keep
# the image under the Trial/Free size limit.
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      dolphin gwenview kali-desktop-kde kate kde-config-gtk-style kdialog kfind \
      kio-extras knewstuff-dialog konsole ksystemstats kwin-addons \
      kwin-style-aurorae kwin-x11 papirus-icon-theme pipewire plasma-desktop \
      plasma-workspace qml-module-qt-labs-platform qt6-style-kvantum \
      qt6-svg-plugins systemsettings wireplumber xdg-desktop-portal \
      xdg-desktop-portal-kde xdg-user-dirs \
      kali-linux-core kali-tools-top10 firefox-esr \
      bind9-dnsutils dirb exploitdb ffuf gobuster whatweb whois wordlists \
 && setcap -r /usr/bin/kwin_wayland \
 && rm -f /usr/share/dbus-1/system-services/org.freedesktop.UDisks2.service \
 && apt-get autoclean \
 && rm -rf /config/.cache /var/lib/apt/lists/* /var/tmp/* /tmp/*

COPY --from=clipboard /out/bin/wl-copy /out/bin/wl-paste /usr/bin/
COPY root/ /
COPY --from=ghcr.io/linuxserver/selkies-layers:amd64-kali-kwin@sha256:dac93cfdabeee7120b0071b3a9710e0f299af97f73073aa65eb4adfbb538a9d0 / /
COPY kali-icon.png /usr/share/selkies/www/icon.png

COPY patch-image.sh /tmp/railway-desktop/patch-image.sh
COPY nginx-origin-map.conf nginx-server-hardening.conf /tmp/railway-desktop/
COPY railway-start.sh /usr/local/bin/railway-start
COPY pkg-restore.sh /usr/local/bin/railway-pkg-restore

# Build-time edits of the image's nginx files. The script checks every
# anchor it edits and fails the build if the upstream files have changed.
RUN bash /tmp/railway-desktop/patch-image.sh \
 && rm -rf /tmp/railway-desktop \
 && chmod 0755 /usr/local/bin/railway-start /usr/local/bin/railway-pkg-restore

EXPOSE 3000

# Railway runs its own init (podman-init) as PID 1, and the image's /init
# (s6-overlay 3.2.1.0) refuses to start unless it is PID 1. railway-start
# brings up the same s6 supervision tree without that check. catatonit
# (part of the linuxserver.io base image) reaps zombies and forwards signals,
# as a child subreaper when it is not PID 1 itself.
ENTRYPOINT ["/usr/bin/catatonit", "--", "/usr/local/bin/railway-start"]
