# Farming Simulator 25 Pterodactyl Egg

A Pterodactyl egg and Docker image for hosting the **Farming Simulator 25 dedicated server** on Linux using Wine.

The container also includes a browser-accessible **noVNC desktop** for installation, activation, and maintenance.

> [!IMPORTANT]
> This repository does **not** contain Farming Simulator 25, DLC, product keys, licenses, or other proprietary GIANTS Software files.
>
> You must provide legally obtained installation files and a valid dedicated-server license.

---

## Features

* Farming Simulator 25 dedicated server support
* Runs the Windows server through Wine
* Browser-based noVNC desktop
* FS25 web administration support
* Automatic Docker image builds through GitHub Actions
* Automatic publishing to GitHub Container Registry
* Automatically generated Pterodactyl egg
* Works from either a personal GitHub account or GitHub organization

---

## Automatic GitHub Builds

Every push to the `main` branch runs the included GitHub Actions workflow.

The workflow automatically:

1. Builds the Docker image from `docker/`
2. Publishes the image to GitHub Container Registry
3. Generates `egg-farming-simulator-25.json`
4. Inserts the correct GHCR image address into the egg
5. Commits the generated egg back to the repository

The published image follows this format:

```text
ghcr.io/OWNER/REPOSITORY:latest
```

Because the repository owner and name are detected automatically, the project can be hosted under either a personal account or an organization without manually editing the egg image URL.

---

## Repository Setup

### 1. Create the repository

Create a new public GitHub repository.

Recommended name:

```text
fs25-pterodactyl
```

Upload all files and folders from this project into the repository root.

Make sure the default branch is:

```text
main
```

### 2. Run the build workflow

Open:

**GitHub Repository → Actions → Build FS25 Pterodactyl Image**

Run the workflow manually, or push a commit to `main`.

GitHub's built-in `GITHUB_TOKEN` is used automatically.

No personal access token or additional repository secret is required.

---

## Make the Container Image Public

After the first successful workflow run:

1. Open the repository on GitHub
2. Open the published package under **Packages**
3. Open **Package settings**
4. Change the package visibility to **Public**

Pterodactyl Wings can pull public GHCR images without registry credentials.

---

## Download the Pterodactyl Egg

The generated egg is available directly from GitHub:

```text
https://raw.githubusercontent.com/OWNER/REPOSITORY/main/egg-farming-simulator-25.json
```

For the Unreal Networks repository:

```text
https://raw.githubusercontent.com/OfficialUnrealNetwork/fs25-pterodactyl/main/egg-farming-simulator-25.json
```

Download it with:

```bash
curl -L \
  https://raw.githubusercontent.com/OfficialUnrealNetwork/fs25-pterodactyl/main/egg-farming-simulator-25.json \
  -o egg-farming-simulator-25.json
```

Then import it through:

**Pterodactyl Admin Panel → Nests → Import Egg**

---

## Required Allocations

Configure the following ports for the Pterodactyl server:

| Port    | Protocol  | Purpose                                  |
| ------- | --------- | ---------------------------------------- |
| `10823` | TCP / UDP | FS25 game traffic and primary allocation |
| `6080`  | TCP       | Browser-based noVNC desktop              |
| `7999`  | TCP       | Farming Simulator web administration     |
| `5900`  | TCP       | Optional direct VNC access               |

Make `10823` the primary Pterodactyl allocation.

---

## First-Time Installation

Create the Pterodactyl server with:

```text
AUTO_INSTALL=true
AUTOSTART_SERVER=false
```

Then:

1. Upload the GIANTS Farming Simulator 25 Windows ESD installer to:

```text
/home/container/installer
```

2. Start the Pterodactyl server.

3. Open the noVNC desktop:

```text
http://NODE-IP:6080/vnc.html?resize=remote&autoconnect=1
```

4. Complete the Farming Simulator 25 installation.

5. Activate the dedicated-server license.

6. Set:

```text
AUTOSTART_SERVER=true
```

7. Restart the Pterodactyl server.

8. Access the FS25 web administration interface on:

```text
http://NODE-IP:7999
```

---

## Updating

To update the Docker image or egg:

1. Modify the Docker, startup, or egg template files.
2. Commit the changes.
3. Push them to `main`.

GitHub Actions will automatically rebuild and publish:

```text
ghcr.io/OWNER/REPOSITORY:latest
```

and regenerate the Pterodactyl egg.

### Pterodactyl Image Cache

Restarting an existing server may continue using a locally cached Docker image.

To ensure Wings uses the latest image, request an image pull through Pterodactyl or recreate the server/container when appropriate.

---

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── build-image.yml
│
├── docker/
│   ├── Dockerfile
│   ├── ptero-entrypoint.sh
│   └── ptero-start.sh
│
├── egg/
│   └── egg-farming-simulator-25.template.json
│
├── scripts/
│   └── render-egg.py
│
└── egg-farming-simulator-25.json
```

### Files

| File                                         | Description                                         |
| -------------------------------------------- | --------------------------------------------------- |
| `egg-farming-simulator-25.json`              | Generated Pterodactyl egg ready for import          |
| `egg/egg-farming-simulator-25.template.json` | Source egg template                                 |
| `docker/Dockerfile`                          | Wine and noVNC container image                      |
| `docker/ptero-entrypoint.sh`                 | Container initialization and entrypoint             |
| `docker/ptero-start.sh`                      | Farming Simulator server startup wrapper            |
| `.github/workflows/build-image.yml`          | GitHub Actions build and publish workflow           |
| `scripts/render-egg.py`                      | Generates the final egg with the correct GHCR image |

---

## Licensing and Game Files

GitHub is only used to host:

* The Pterodactyl egg
* Docker configuration
* Startup scripts
* Supporting project files
* The generated container image

Licensed Farming Simulator 25 files must remain inside the Pterodactyl server's persistent storage.

Do **not** upload the FS25 installer, activated server files, product keys, DLC, or other proprietary GIANTS Software content to this repository or any public container image.

---

## Disclaimer

This project is an unofficial community project and is not affiliated with, endorsed by, or supported by GIANTS Software.

Farming Simulator and related trademarks are property of their respective owners.
