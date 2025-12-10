# Docker Container Commands

This guide provides a comprehensive reference for managing Docker containers. It covers the essential commands for the entire container lifecycle, from creation and execution to inspection and removal.

## Container Lifecycle Overview

1.  **Create/Run**: Instantiate a container from an image (`docker run`).
2.  **Manage**: Start, stop, or restart existing containers (`docker start`, `stop`, `restart`).
3.  **Inspect**: View logs, processes, and metadata (`docker logs`, `top`, `inspect`).
4.  **interact**: Execute commands inside a running container (`docker exec`, `attach`).
5.  **Cleanup**: Remove stopped containers to free resources (`docker rm`, `prune`).

---

## 1. Docker Run (`docker run`)

The `docker run` command is the primary way to start new containers. It creates a writeable container layer over the specified image and starts usage instructions.

**Syntax:**
```bash
docker run [options] <image> [command] [args...]
```

### Common Options

| Category | Option | Description |
| :--- | :--- | :--- |
| **General** | `-d`, `--detach` | Run container in background and print container ID. |
| | `--name <string>` | Assign a custom name to the container. |
| | `--rm` | Automatically remove the container when it exits. |
| | `-it` | Combine `-i` (interactive) and `-t` (pseudo-TTY) for interactive shell access. |
| **Networking** | `-p`, `--publish <host>:<container>` | Publish a container's port to the host. |
| | `-P`, `--publish-all` | Publish all exposed ports to random ports. |
| | `--network <network>` | Connect a container to a network. |
| **Storage** | `-v`, `--volume <host>:<container>` | Bind mount a volume. |
| **Environment** | `-e`, `--env <key=value>` | Set environment variables. |
| | `--env-file <file>` | Read in a file of environment variables. |
| **Resources** | `--cpus <decimal>` | Number of CPUs to limit the container to. |
| | `--memory <bytes>` | Memory limit (e.g., `512m`, `1g`). |
| **Policy** | `--restart <policy>` | Restart policy (`no`, `on-failure`, `always`, `unless-stopped`). |

### Examples

**Run a web server in the background:**
```bash
docker run -d --name my-nginx -p 8080:80 nginx
```
*Access at `http://localhost:8080`.*

**Run an interactive Ubuntu shell:**
```bash
docker run -it ubuntu /bin/bash
```

**Run with environment variables and auto-removal:**
```bash
docker run --rm -e DEBUG=true -e API_KEY=secret_key my-app:latest
```

**Run with restart policy and volume mount:**
```bash
docker run -d \
  --name database \
  --restart always \
  -v db_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:5.7
```

---

## 2. listing Containers (`docker ps`)

List containers to check their status, ID, names, and port mappings.

**Syntax:**
```bash
docker ps [options]
```

### Options

| Option | Description |
| :--- | :--- |
| `-a`, `--all` | Show all containers (default shows just running). |
| `-q`, `--quiet` | Only display container IDs (useful for scripting). |
| `-s`, `--size` | Display total file sizes. |
| `--format <string>` | Pretty-print containers using a Go template. |
| `-l`, `--latest` | Show the latest created container (includes all states). |

### Examples

**List all running containers:**
```bash
docker ps
```

**List all containers (running and stopped):**
```bash
docker ps -a
```

**Get only the IDs of all containers:**
```bash
docker ps -a -q
```
*Usage tip: `docker rm $(docker ps -a -q)` removes all containers.*

---

## 3. Stopping and Starting

Manage the runtime state of existing containers.

### `docker stop`
Stops one or more running containers gracefully (sends `SIGTERM`, then `SIGKILL` after grace period).

**Syntax:**
```bash
docker stop [options] <container_name_or_id>
```

**Examples:**
```bash
docker stop my-nginx
docker stop $(docker ps -q)  # Stop all running containers
```

### `docker start`
Starts one or more stopped containers.

**Syntax:**
```bash
docker start [options] <container_name_or_id>
```

**Options:**
*   `-a`, `--attach`: Attach STDOUT/STDERR and forward signals.
*   `-i`, `--interactive`: Attach container's STDIN.

**Examples:**
```bash
docker start my-nginx
```

### `docker restart`
Restarts one or more containers (stop then start).

**Syntax:**
```bash
docker restart [options] <container_name_or_id>
```

**Examples:**
```bash
docker restart my-web-app
```

---

## 4. Removing Containers (`docker rm`)

Remove one or more containers. You generally need to stop a container before removing it, or use `--force`.

**Syntax:**
```bash
docker rm [options] <container_name_or_id>
```

### Options

| Option | Description |
| :--- | :--- |
| `-f`, `--force` | Force the removal of a running container (uses SIGKILL). |
| `-v`, `--volumes` | Remove anonymous volumes associated with the container. |

### Examples

**Remove a specific container:**
```bash
docker rm my-old-container
```

**Force remove a running container:**
```bash
docker rm -f my-nginx
```

**Remove all stopped containers (Prune):**
```bash
docker container prune
```
*Prompts for confirmation. Removes all stopped containers.*

---

## 5. Inspection & Logs

Debugging tools to understand what is happening inside your containers.

### `docker logs`
Fetch the logs of a container.

**Syntax:**
```bash
docker logs [options] <container>
```

**Options:**
*   `-f`, `--follow`: Follow log output (like `tail -f`).
*   `--tail <n>`: Number of lines to show from the end of the logs.
*   `-t`, `--timestamps`: Show timestamps.

**Examples:**
```bash
docker logs -f my-app
docker logs --tail 100 my-database
```

### `docker inspect`
Return low-level information on Docker objects (JSON format).

**Syntax:**
```bash
docker inspect [options] <name_or_id>
```

**Examples:**
```bash
docker inspect my-nginx
```
*Tip: Use `grep` or `--format` to extract specific info (IP address, mounts).*

### `docker top`
Display the running processes of a container.

**Syntax:**
```bash
docker top <container>
```

### `docker stats`
Display a live stream of container resource usage statistics (CPU, generic, network I/O).

**Syntax:**
```bash
docker stats [options] [container...]
```

---

## 6. Execution (`docker exec`)

Run a command inside a running container. This is primarily used for debugging or administrative tasks.

**Syntax:**
```bash
docker exec [options] <container> <command> [args...]
```

### Options

| Option | Description |
| :--- | :--- |
| `-d`, `--detach` | Detached mode: run command in the background. |
| `-i`, `--interactive` | Keep STDIN open even if not attached. |
| `-t`, `--tty` | Allocate a pseudo-TTY. |
| `-u`, `--user` | Username or UID (format: `<name\|uid>[:<group\|gid>]`). |
| `-w`, `--workdir` | Working directory inside the container. |

### Examples

**Open a bash shell in a running container:**
```bash
docker exec -it my-container bash
```
*(Use `sh` if `bash` is not installed)*

**Run a database backup command:**
```bash
docker exec my-postgres pg_dump dbname > backup.sql
```

**Check a file specifically:**
```bash
docker exec my-app cat /var/log/app.log
```

---

## Summary Cheat Sheet

| Command | Action | Key Options |
| :--- | :--- | :--- |
| `docker run` | Create & start container | `-d`, `-p`, `-v`, `-e`, `--name` |
| `docker ps` | List containers | `-a`, `-q` |
| `docker stop` | Stop container | |
| `docker start` | Start stopped container | `-a` |
| `docker rm` | Remove container | `-f` (force) |
| `docker logs` | View logs | `-f` (follow) |
| `docker exec` | Run command inside | `-it` (interactive shell) |