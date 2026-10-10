# Farming Simulator 25 Pterodactyl Egg

<p align="center">
  <img src="https://i.imgur.com/OBRgCqw.png" alt="Farming Simulator 25 Pterodactyl Egg" width="100%">
</p>

A Pterodactyl egg and Docker image for running the **Farming Simulator 25 dedicated server** on Linux. The Windows server runs under Wine, and a browser desktop (noVNC) is included so you can run the GIANTS installer and activate your server licence without ever touching the node.

> [!IMPORTANT]
> This repository does **not** include Farming Simulator 25, DLC, product keys or any other GIANTS Software files. You need your own legally obtained FS25 installer and a valid dedicated-server licence.

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Quick start](#quick-start)
- [Installing FS25 on the server](#installing-fs25-on-the-server)
- [Startup settings](#startup-settings)
- [Server files and logs](#server-files-and-logs)
- [Updating](#updating)
- [Troubleshooting](#troubleshooting)
- [Building your own image](#building-your-own-image)
- [Credits](#credits)
- [Licence](#licence)

## Features

- Runs the Windows FS25 dedicated server on a Linux node through Wine.
- Browser desktop (noVNC) on port `6080` for the GIANTS installer and licence activation.
- Automatic installer launch: upload the ESD image, start the server, and the setup opens in the browser desktop. The image is unpacked inside the container, so there's no need to extract it yourself.
- Server name, passwords, player count, map, difficulty, crossplay and more are set from the Pterodactyl **Startup** tab and applied on every restart.
- FS25 web administration panel on port `7999`, with optional dark mode.
- Clean console by default: a status banner with your admin, join and noVNC links, followed by the live FS25 game log. Switch it off to see every raw Wine, VNC and web log line.
- All game files, saves and settings live in the server's persistent volume, so they survive restarts and image updates.

## Requirements

- A working **Pterodactyl Panel** and **Wings** node.
- An **x86_64 (amd64)** node. The image is built for `linux/amd64` only.
- The **Farming Simulator 25 Windows ESD installer** from your GIANTS account, plus a **dedicated-server licence key**. The Steam edition is not supported by this install flow.
- These ports free on the node IP you assign:

  | Port    | Protocol  | Purpose                                          |
  | ------- | --------- | ------------------------------------------------ |
  | `10823` | TCP + UDP | Game traffic. Use this as the primary allocation |
  | `6080`  | TCP       | noVNC browser desktop                            |
  | `7999`  | TCP       | FS25 web administration panel                    |
  | `5900`  | TCP       | Optional, direct VNC access                      |

- Enough disk space for the installer image, its unpacked copy and the installed game at the same time. Be generous: the install fails if the disk fills up part way through.
- Enough memory for the FS25 dedicated server plus Wine and the desktop.

## Quick start

1. **Download the egg** from this repository:

   ```bash
   curl -L -o egg-farming-simulator-25.json \
     https://raw.githubusercontent.com/Unreal-Network/FS25-Pterodactyl-Egg/main/egg-farming-simulator-25.json
   ```

2. **Import it.** In the Pterodactyl admin area go to **Nests**, choose a nest (or create one, for example "Farming Simulator"), click **Import Egg** and select the file.

3. **Add the allocations** to your node: `10823`, `6080` and `7999` (and `5900` if you want direct VNC) on the IP you will use.

4. **Create the server** with the *Farming Simulator 25 Wine noVNC* egg:
   - Primary allocation: `10823`.
   - Additional allocations: `6080` and `7999` (and `5900` if wanted).
   - Give it plenty of disk and memory (see [Requirements](#requirements)).
   - Leave **Auto-install** on `true` and **Server startup mode** on `false` for the first boot.
   - Change every password that starts with `ChangeMe`.
   - Set **Public hostname** to the hostname or IP players use to reach your node.

5. Follow [Installing FS25 on the server](#installing-fs25-on-the-server).

The egg's install script only creates the folder layout and an `UPLOAD_INSTALLER_HERE.txt` reminder. It never downloads any game or licence files.

## Installing FS25 on the server

1. **Upload the installer.** Using the panel's file manager or SFTP, upload the GIANTS ESD image into the `installer` folder. Keep the original file name (`FarmingSimulator25_<version>_ESD.img`), because the automatic installer looks for that pattern. Keep only one installer archive in the folder.

2. **Start the server.** About 12 seconds after the desktop comes up, the installer starts on its own. It unpacks the image first, which can take a while.

3. **Open the browser desktop** at:

   ```text
   http://YOUR-HOST:6080/vnc.html?resize=remote&autoconnect=1
   ```

   Sign in with your **noVNC password**.

4. **Install FS25 and activate your dedicated-server licence** in the installer windows that appear on the desktop.

5. **Switch the server on.** In the **Startup** tab set **Server startup mode** to `true`, then restart the server.

6. **Open the web admin** at `http://YOUR-HOST:7999` and sign in with your **web admin username and password**. From there you can start and manage the game server.

Once setup has created the server's configuration, the installer is not launched again on later restarts. You can then delete the installer image to free up disk space.

## Startup settings

Everything below is in the server's **Startup** tab. The game and web admin settings are written into the FS25 configuration files on every restart, so the Startup tab is the place to change them.

| Setting | Variable | Default | What it does |
| --- | --- | --- | --- |
| Server name | `SERVER_NAME` | `My FS25 Server` | Name shown in the FS25 server browser |
| Game password | `SERVER_PASSWORD` | *(empty)* | Password to join. Leave empty for no password |
| In-game admin password | `SERVER_ADMIN` | `ChangeMe-Admin` | Password to sign in as admin inside the game |
| Maximum players | `SERVER_PLAYERS` | `8` | 1 to 16 |
| Map ID | `SERVER_MAP` | `MapUS` | FS25 map ID |
| Difficulty | `SERVER_DIFFICULTY` | `3` | Economic difficulty, 1 to 3 |
| Language/region | `SERVER_REGION` | `en` | Server language/region code |
| Crossplay | `SERVER_CROSSPLAY` | `true` | Allow PC and console players |
| Pause behavior | `SERVER_PAUSE` | `2` | Dedicated-server pause mode, 0 to 2 |
| Save interval | `SERVER_SAVE_INTERVAL` | `180.000000` | Autosave interval value passed to the server |
| Statistics interval | `SERVER_STATS_INTERVAL` | `31536000` | Statistics interval value passed to the server |
| Web admin username | `WEB_USERNAME` | `admin` | Web admin login on port 7999 |
| Web admin password | `WEB_PASSWORD` | `ChangeMe-Web` | Web admin password on port 7999 |
| Dark web admin | `WEB_DARKMODE` | `false` | Dark theme for the web admin |
| noVNC password | `VNC_PASSWORD` | `ChangeMe-VNC` | Password for the browser desktop |
| Browser title | `WEBPAGE_TITLE` | `Farming Simulator 25 Server` | Title of the noVNC browser tab |
| Auto-install | `AUTO_INSTALL` | `true` | Launch the installer when an ESD image is found and setup is not finished |
| Server startup mode | `AUTOSTART_SERVER` | `false` | `false`: desktop only. `true`: game and web admin. `web_only`: web admin only |
| Pretty console | `PRETTY_LOGS` | `true` | `true`: clean status console. `false`: every raw log line |
| Public hostname | `PUBLIC_HOST` | `node1.unrealcorp.net` | Hostname used in the console's quick links. **Change this to your own** |

The game port comes from the server's primary allocation, so you don't set it here.

## Server files and logs

| Path (in the file manager) | Contents |
| --- | --- |
| `installer/` | Upload the GIANTS ESD image here |
| `dlc/` | DLC files |
| `game/Farming Simulator 2025/` | The installed game, including `dedicatedServer.xml` (web admin settings) |
| `config/FarmingSimulator2025/` | Game profile and the FS25 game logs (`log_*.txt`) |
| `config/FarmingSimulator2025/dedicated_server/dedicatedServerConfig.xml` | Game server settings |
| `config/FarmingSimulator2025/pterodactyl-startup.log` | Container startup log (Wine, VNC, web) when the pretty console is on |
| `setup-logs/setup.log` | Output of the automatic installer |
| `.fs25server/` | Wine prefix |

## Updating

**Server image.** The egg uses `ghcr.io/officialunrealnetwork/fs25-pterodactyl:latest`. If a server is still running an older build after an update, pull the image on the node and restart the server:

```bash
docker pull ghcr.io/officialunrealnetwork/fs25-pterodactyl:latest
```

Your game files, saves and settings stay in the server volume.

**Egg.** Download the latest `egg-farming-simulator-25.json` and import it again over the existing egg in the panel.

## Troubleshooting

**The installer never starts.**
Check that **Auto-install** is `true` and that the image in `installer/` still has its original `FarmingSimulator25_<version>_ESD.img` name. The installer only runs while setup is not finished, and waits about 12 seconds after the desktop comes up. Its output is in `setup-logs/setup.log`.

**"More than one FS25 installer archive was found".**
Keep exactly one installer image in `installer/` and delete the others.

**The installer stops while unpacking.**
The disk is usually full. Raise the server's disk limit, delete the half-unpacked files in `installer/` (keep the `.img`), and restart.

**I can't reach noVNC or the web admin.**
Make sure `6080` and `7999` are added to the server as allocations on the same IP as the game port, and that your firewall allows them. These two ports are fixed, so each node IP can host one FS25 server at a time.

**The console links point to the wrong address.**
Set **Public hostname** in the Startup tab to your node's hostname or IP, then restart.

**The panel says the server is running, but the game is offline.**
That's expected. The panel marks the server as running once the container is up. The game itself starts when **Server startup mode** is `true`, or when you click **Start** in the web admin on port 7999.

**My changes in the web admin or the XML files keep resetting.**
Server name, passwords, player count, map, difficulty, crossplay, language, intervals, pause mode and the web admin login are written from the Startup tab on every restart. Change those in the Startup tab instead.

**`[FS25 CONFIG] ERROR: Farming Simulator 25 installation was not found`.**
The game is not installed yet. Finish [Installing FS25 on the server](#installing-fs25-on-the-server) first.

**I need more detail in the console.**
Set **Pretty console** to `false` and restart to see every raw log line, or read `config/FarmingSimulator2025/pterodactyl-startup.log`.

## Building your own image

Every push to `main` (except changes to the README or the generated egg) runs the GitHub Actions workflow in `.github/workflows/build-image.yml`. It:

1. Builds the image from `docker/` and publishes it to `ghcr.io/<owner>/<repository>:latest`.
2. Renders `egg/egg-farming-simulator-25.template.json` into `egg-farming-simulator-25.json` with that image address, using `scripts/render-egg.py`.
3. Commits the updated egg back to the repository.

No extra secrets are needed; the built-in `GITHUB_TOKEN` is used. GitHub publishes new packages as private, so after the first build open the package's settings on GitHub and set its visibility to **Public**, or Wings won't be able to pull it.

To customise the egg, edit the template in `egg/`, not the generated JSON.

```text
.
├── .github/workflows/build-image.yml      Build, publish and egg generation
├── docker/
│   ├── Dockerfile                         Image build
│   ├── console-wrapper.sh                 Pretty console and quick links
│   ├── ptero-entrypoint.sh                Container entrypoint
│   ├── ptero-start.sh                     Startup and automatic installer launch
│   ├── ptero-setup-fs25.sh                Finds and unpacks the ESD installer
│   ├── copy_server_config.sh              Creates the FS25 config files if missing
│   └── apply-pterodactyl-config.py        Applies Startup settings to the config files
├── egg/egg-farming-simulator-25.template.json
├── scripts/render-egg.py
└── egg-farming-simulator-25.json          Ready-to-import egg
```

## Credits

Created and maintained by [Mkkbeydoun](https://github.com/Mkkbeydoun) for Unreal Network.

The image is built on top of [`toetje585/arch-fs25server`](https://hub.docker.com/r/toetje585/arch-fs25server), which provides the Wine, noVNC and FS25 web server base. This project adapts it for Pterodactyl.

This is an unofficial community project. It is not affiliated with, endorsed by or supported by GIANTS Software. Farming Simulator and related trademarks belong to their respective owners.

## Licence

The files in this repository are released under the [MIT Licence](LICENSE). This covers this project's own files only, not the base image, Farming Simulator 25 or any GIANTS Software content.
