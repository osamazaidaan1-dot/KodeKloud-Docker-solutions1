# TASK1 : Install Docker Packages and Start Docker Service

## 🎯 Task

The Nautilus DevOps team plans to containerize applications on **App Server 1 (stapp01)**. Install Docker, Docker Compose, and ensure the Docker service is running and enabled.

---

## 🛠️ Steps

1. **SSH into App Server 1**

```bash
ssh tony@stapp01
```

2. **Enable Docker repository (CentOS/RHEL)**

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3. **Install Docker Engine and components**

```bash
sudo dnf install -y \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

4. **Start and enable Docker**

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

5. **Verify installation**

```bash
docker --version
docker compose version
sudo systemctl status docker --no-pager
```

---


## ✅ Notes

- Requires sudo privileges for `dnf` and `systemctl`.
- Use `sudo usermod -aG docker <user>` if running Docker without sudo is desired.
- For offline nodes, mirror Docker RPMs from the official repo.

## ✅ Explanation

Here is the step-by-step breakdown of your command history, organized by phase with the core concepts behind each command:

---

### Phase 1: Initial Discovery & OS Identification

#### `1. docker version` & `2. docker --version`

* **Command:** Checks if the Docker binary is present in the system's `$PATH` and queries its version.
* **Concept:**
* `docker --version` queries just the client binary.
* `docker version` queries both the **Client** (the CLI you type into) and the **Server/Daemon** (`dockerd`, the background process managing containers). If Docker is not installed, the shell returns `command not found`.


#### `3. sudo systemctl status docker`

* **Command:** Asks `systemd` (the Linux init and service manager) whether the background service named `docker.service` exists and is running.
* **Concept:** Docker operates as a client-server architecture. Even if the CLI binary exists, the daemon must run as a system service to create containers, networks, and storage volumes.

#### `4. sudo apt-get update`

* **Command:** Attempts to refresh the package cache using the Debian/Ubuntu package manager (`apt`).
* **Concept:** This command likely failed or returned `apt-get: command not found` because the server is not a Debian/Ubuntu-based system, triggering the need to check the OS in the next step.

#### `5. cat /etc/os-release`

* **Command:** Reads the standard Linux file that contains operating system identification data.
* **Concept:** Modern Linux distributions follow the `systemd` standard of defining distribution name, version ID, and family inside `/etc/os-release`. Inspecting this revealed that the server runs an **RHEL/CentOS/Fedora-family** distribution (which uses `dnf`/`rpm` instead of `apt`/`dpkg`).

---

### Phase 2: Repository Setup & Package Installation

#### `6. dnf -f install dnf-plugins-core` & `7. dnf -y install dnf-plugins-core`

* **Command:** Installs core extension plugins for `dnf` (the package manager used in CentOS/RHEL 8+ and Fedora).
* **Concept:**
* Command 6 failed because `-f` is not a valid parameter here.
* Command 7 used `-y` (automatically answer "yes" to prompts).
* `dnf-plugins-core` provides utility modules, most notably `dnf config-manager`, which allows you to safely add, enable, or disable external package repositories.



#### `8. dnf config-manager --add-repo [https://download.docker.com/linux/centos/docker-ce.repo](https://download.docker.com/linux/centos/docker-ce.repo)`

* **Command:** Downloads Docker's official `.repo` file into `/etc/yum.repos.d/`.
* **Concept:** Default enterprise Linux repositories often ship older, fork packages (like `podman` or older `docker`). Adding the official Docker repository tells `dnf` where to download the authentic, up-to-date Community Edition (`docker-ce`) binaries.

#### `9. dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

* **Command:** Installs the core components of the Docker engine and its tools.
* **Concept (Package Breakdown):**
* `docker-ce`: The core Docker daemon (`dockerd`) that manages containers, images, and networks.
* `docker-ce-cli`: The command-line client (`docker`) used to send API requests to the daemon.
* `containerd.io`: The low-level container runtime (CRI-compliant) that directly manages the Linux kernel namespaces, cgroups, and container lifecycles.
* `docker-buildx-plugin`: Docker's modern build tool leveraging BuildKit for parallel and multi-architecture image builds.
* `docker-compose-plugin`: Installs modern **Compose V2** (accessed via `docker compose`), replacing the legacy Python-based Compose V1 (`docker-compose`).



---

### Phase 3: Service Management (systemd)

#### `10. systemctl status docker`

* **Command:** Checks service state immediately after installation.
* **Concept:** On RHEL/CentOS systems, package installation does **not** automatically start or enable services. The status here would show `inactive (dead)`.

#### `11. systemctl start docker`

* **Command:** Starts the `docker` daemon process immediately in the current running session.
* **Concept:** `systemd` spins up `dockerd` and opens the Unix socket (`/var/run/docker.sock`) through which the CLI communicates with the engine.

#### `12. systemctl status docker`

* **Command:** Checks service state after starting.
* **Concept:** Confirms the daemon successfully initialized, showing `Active: active (running)`.

#### `13. systemctl enable docker`

* **Command:** Configures the Docker service to automatically launch upon system boot.
* **Concept:** Creates a symbolic link in `/etc/systemd/system/multi-user.target.wants/` pointing to the Docker service unit file. *(Note: `systemctl enable --now docker` combines steps 11 and 13 into one).*

#### `14. systemctl status docker`

* **Command:** Final service verification.
* **Concept:** Confirms that the service is both `Loaded: ... (enabled; ...)` and `Active: active (running)`.

---

### Phase 4: Final Validation

#### `15. docker version`

* **Command:** Queries both the Docker CLI and the active server daemon.
* **Concept:** Because the daemon is now running, this command successfully outputs:
* **Client:** Version, API version, OS/Arch.
* **Server (Engine):** Version, storage driver (`overlay2`), containerd version, runc version.



#### `16. docker compose version`

* **Command:** Confirms that the Compose V2 plugin is detected and functional.
* **Concept:** Validates that the multi-container orchestration tool is available for running multi-service stacks defined in `docker-compose.yml`.
