# Deploy and Host Kali Linux KDE Desktop with Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/kali-linux-20263-kde-desktop-in-your-bro?referralCode=20H68q)

A Kali Linux (kali-rolling) desktop with KDE Plasma that you open in a browser tab. It comes with Firefox, Dolphin and Konsole, Kali's top 10 tools (`kali-tools-top10`: Metasploit, Burp Suite, nmap, sqlmap, Hydra, John the Ripper, Wireshark, Aircrack-ng, NetExec, Responder) and gobuster, ffuf, dirb, whatweb, searchsploit and the Kali wordlists. More Kali packages are one variable away. A generated password protects it, and your home directory (files, settings, browser profile) lives on a persistent volume.

## About Hosting Kali Linux KDE Desktop

A browser desktop needs a display server, a desktop session, a streaming server, a login in front of it, and storage that outlives rebuilds. This community template builds Kali's KDE desktop the way linuxserver.io builds its `kali-linux` image, on the same base (`baseimage-selkies`, pinned by digest), and streams it with Selkies over a single HTTPS WebSocket. The tool set is smaller than linuxserver.io's so the image fits Railway's Trial and Free plans. Railway runs its own init as PID 1, which the image's s6-overlay init refuses to run under, so a small start script brings up the image's own services under s6 supervision instead. nginx requires a login on every page except `/healthz`. `/config` (the home directory) is a Railway volume.

## Common Use Cases

- Work in a full Linux environment from any device with a browser, including managed work laptops
- Security training, CTFs and lab work against targets you own or are allowed to test
- Open untrusted links or files in a cloud browser instead of on your own computer
- A ready Linux environment for training sessions, demos and support work

## Dependencies for Kali Linux KDE Desktop Hosting

- One service built from a Dockerfile on top of `ghcr.io/linuxserver/baseimage-selkies` (kali), with Kali packages from the official archive (no database, no external API keys)
- A Railway volume mounted at `/config` for the home directory

### Deployment Dependencies

- [linuxserver/docker-kali-linux](https://github.com/linuxserver/docker-kali-linux)
- [linuxserver/docker-baseimage-selkies](https://github.com/linuxserver/docker-baseimage-selkies)
- [Selkies configuration reference](https://docs.linuxserver.io/selkies/user-guide/configuration/)
- [Selkies web client guide (sidebar, clipboard, files)](https://docs.linuxserver.io/selkies/user-guide/web-client/)
- [Kali Linux documentation](https://www.kali.org/docs/)

### Implementation Details

| Service | What runs inside | Volume | Port |
|---|---|---|---|
| Desktop | Wayland (KWin) desktop session, D-Bus, audio, Selkies (loopback only), nginx (login, `/healthz`), all supervised by s6 | `/config` | 3000 (public HTTPS domain) |

**Plan:** runs on Trial and up. The image stays under the 4 GB image size limit of Trial and Free. Measured on Railway with a 1 GB memory limit (Trial): about 0.75 GB of RAM with one browser session connected and no apps open. Free's 0.5 GB is too small for KDE. Burp Suite, Metasploit and Firefox with several tabs need Hobby and 2 GB or more.

**First login**

1. Wait for the deploy to turn healthy. The first build downloads about 1.7 GB of Kali packages, so allow about 10 minutes.
2. Open the public URL. The browser asks for a login.
3. User `kali` (or your `CUSTOM_USER`; letters, digits and `. _ @ -` only, otherwise it stays `kali`), password = `PASSWORD` from the service Variables tab.
4. The desktop appears and resizes to your browser window. The Selkies side menu has the clipboard box, file upload and download, audio, keyboard and display settings.

**Authorized use only**

Kali's tools can scan, brute-force and exploit systems. Use them only against systems you own or have written permission to test. Railway's [Fair Use Policy](https://railway.com/legal/fair-use) and [Terms of Service](https://railway.com/legal/terms) apply to everything that runs in this desktop, and scanning or attacking third-party networks from Railway can get your account suspended. Raw-socket tools that need `NET_RAW`/`NET_ADMIN` (for example `nmap -sS`, packet capture or Wi-Fi tools) may not work in Railway's containers; `nmap -sT` and HTTP-level tools do.

**Security**

- The desktop never runs without a login. Basic auth is on in nginx at build time, and the service refuses to start if `PASSWORD` is empty or shorter than 8 characters (use 12 or more).
- Only a hash of the password is stored, and the password is removed from the environment before any desktop app starts.
- Requests from other websites are refused (Origin check in nginx and Selkies), so a page you visit cannot use your cached login to drive the desktop or upload files.
- The Linux user `abc` has passwordless sudo. Anyone with the password controls the whole container. The image's `HARDEN_DESKTOP=true` option disables sudo, terminals and the open-with tools for a locked-down desktop.
- Browsers keep basic-auth logins until they close. Use a private window on shared computers.

**What persists**

Everything in `/config`: documents, `~/Desktop`, desktop settings, browser profiles. Packages installed with `sudo apt install` live in the image and reset on every redeploy. List them in the `APT_PACKAGES` variable or in `~/.config/railway-desktop/packages` (one or more per line) and they are reinstalled in the background after each start (log: `/var/log/railway-desktop/pkg-restore.log`).

**No GPU: software rendering**

Railway has no GPUs. The desktop is drawn on the CPU (Mesa's software renderer), and Selkies encodes the video stream on the CPU. Office work, browsing and development tools run well; 3D apps, games and video editing will be slow. A busy screen costs CPU: lower the frame rate in the side menu, or set `SELKIES_FRAMERATE`, to save CPU.

**Audio and clipboard**

- Sound from desktop apps streams to your browser over the same WebSocket. Browsers only play it after you click in the page once.
- Text copy and paste works both ways. Chromium-based browsers ask for clipboard permission. In Firefox and Safari, paste through the clipboard box in the side menu.

**Scaling and versions**

The desktop is a single-user, volume-backed service: run one replica and scale it vertically. Every build installs the current kali-rolling packages. To move to a newer base, change the digest on the `FROM` line in `services/desktop/Dockerfile` (the `kali` tag of `ghcr.io/linuxserver/baseimage-selkies`) and redeploy. The home directory is kept.

**Limits**

No Docker inside the desktop (Railway runs no privileged containers), no WebRTC/UDP (the WebSocket stream is used instead), and running apps restart on every redeploy. Railway's app sleeping stops the desktop session as well. Use the desktop for your own work: the Fair Use Policy forbids cryptocurrency mining, torrenting, reselling compute, operating proxies or anonymization services, and abusive bots or scrapers.

### Why Deploy Kali Linux KDE Desktop on Railway?

Railway gives the desktop HTTPS, a persistent volume and usage-based billing without any server to manage. You get a Linux desktop at a URL in a few minutes, pay for the RAM and CPU it actually uses, and can resize or redeploy it from the dashboard. This is a community template, not affiliated with linuxserver.io, the Selkies project, the distribution's publishers or Railway.
