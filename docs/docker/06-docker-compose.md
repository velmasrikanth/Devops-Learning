# Docker Compose

**Docker Compose** is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services, networks, and volumes. Then, with a single command, you create and start all the services from your configuration.

## Key Concepts

*   **Service**: A container definition (like a `docker run` command) that scales across one or more containers.
*   **Volume**: Persistent storage shared between containers or the host.
*   **Network**: Communication channel between containers.

---

## 1. The `docker-compose.yaml` File

The core of Docker Compose is the `docker-compose.yaml` file. It defines the desired state of your application stack.

### Structure Breakdown

```yaml
version: "3.8"  # (Optional in recent versions)

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
  
  database:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secret

volumes:
  db-data:

networks:
  frontend:
```

### Common Service Configuration Keys

| Key | Description | Example |
| :--- | :--- | :--- |
| `image` | Docker image to use. | `image: nginx` |
| `build` | Path to a Dockerfile to build custom image. | `build: .` or `build: ./backend` |
| `ports` | Port mapping (Host:Container). | `ports: ["8080:80"]` |
| `environment` | Environment variables. | `environment: ["DEBUG=1"]` |
| `volumes` | Mount host paths or named volumes. | `volumes: ["./data:/app/data"]` |
| `networks` | Networks to join. | `networks: ["backend"]` |
| `depends_on` | Start order dependency. | `depends_on: ["database"]` |
| `restart` | Restart policy. | `restart: always` |
| `command` | Override the default command. | `command: python manage.py runserver` |


---

## 2. Detailed Service Configuration

The `services` block is the most critical part of the Compose file. It defines the containers that will run.

### `build`
Defines how to build the image from source code. Learn more about [Docker Build](03-docker-images.md).

*   **`context`**: The path to the directory containing the Dockerfile (relative to the compose file).
*   **`dockerfile`**: Alternate name for the Dockerfile (default is `Dockerfile`).
*   **`args`**: Build arguments to pass to the build process.

```yaml
services:
  webapp:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
      args:
        - APP_VERSION=1.0
```

### `image`
Specifies the image to start the container from.
*   If the image doesn't exist locally, Compose attempts to pull it.
*   If `build` is also specified, `image` defines the name/tag for the built image.

```yaml
image: nginx:alpine                  # Pull from hub
image: my-registry.com/app:v1        # Private registry
```

### `ports`
Exposes container ports to the host machine.

*   **Short Syntax**: `"HOST_PORT:CONTAINER_PORT"`
*   **Ranges**: `"8000-8010:8000-8010"`
*   **Long Syntax**: Allows defining protocol (tcp/udp) and mode.

```yaml
ports:
  - "3000:3000"             # Map host 3000 to container 3000
  - "127.0.0.1:8080:80"     # Limit binding to localhost
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

### `environment` & `env_file`
Define environment variables inside the container.

*   **`environment`**: Inline key-value pairs (array or map syntax).
*   **`env_file`**: Path to a file containing variables (`KEY=VAL`).

```yaml
environment:
  - DEBUG=true
  - DB_HOST=postgres
env_file:
  - .env.production
```

### `volumes` (Service Level)
Mounts host paths or named volumes into the service.

*   **Short Syntax**: `"SOURCE:TARGET[:MODE]"`
    *   `SOURCE`: Host path or Named Volume.
    *   `TARGET`: Path inside the container.
    *   `MODE`: `ro` (read-only) or `rw` (read-write).

```yaml
volumes:
  - ./code:/app             # Bind mount (updates live)
  - db-data:/var/lib/mysql  # Named volume (persistent)
  - /opt/data:/app/data:ro  # Read-only bind mount
```

### `networks` (Service Level)
Defines which networks the service joins. Services on the same network can reach each other by service name (DNS).

```yaml
services:
  web:
    networks:
      - frontend
      - backend
```

---

## 3. Volumes and Networks Explained

### Volumes in Detail
Volumes are the preferred mechanism for persisting data generate by and used by Docker containers.

1.  **Named Volumes**:
    *   Managed completely by Docker.
    *   Best for database storage or data shared between containers.
    *   Defined in the top-level `volumes:` key.

2.  **Bind Mounts**:
    *   Maps a specific file or directory on the host to the container.
    *   Best for development (sharing source code).

**Top-Level Volume Definition:**
```yaml
volumes:
  db-data:
    driver: local             # Default driver
    driver_opts:              # Advanced options (e.g. NFS)
      type: none
      device: /path/to/dir
      o: bind
```

### Networks in Detail
Networks determine how containers communicate with each other and the outside world.

*   **`bridge`**: The default driver. detailed isolation.
*   **`host`**: Removes network isolation. The container shares the host's networking namespace.
*   **`overlay`**: Used for Swarm services to communicate across multiple nodes.
*   **`none`**: Disables networking.

**Top-Level Network Definition:**
```yaml
networks:
  app-net:
    driver: bridge
    ipam:                     # Custom IP Address Management
      config:
        - subnet: 172.16.238.0/24
```

---


## 4. Docker Compose CLI Commands

The `docker-compose` (or `docker compose` in V2) CLI is used to manage the lifecycle of the application defined in the YAML file.

### Management Commands

| Command | Description | Key Options |
| :--- | :--- | :--- |
| `docker-compose up` | Build, (re)create, start, and attach to containers. | `-d` (detached), `--build` (force rebuild) |
| `docker-compose down` | Stop and remove resources (containers, networks). | `-v` (remove volumes as well) |
| `docker-compose start` | Start existing containers. | |
| `docker-compose stop` | Stop running containers. | |
| `docker-compose restart` | Restart containers. | |

### Inspection & Interaction

| Command | Description |
| :--- | :--- |
| `docker-compose ps` | List containers and their status. |
| `docker-compose logs` | View output from containers. (Use `-f` to follow). |
| `docker-compose exec` | Run a command in a running container. |
| `docker-compose config` | Validate and view the Compose file. |

**Examples:**
```bash
# Start all services in the background
docker-compose up -d

# Stop all services and remove the volumes
docker-compose down -v

# View logs for a specific service
docker-compose logs -f web

# Run a shell inside the 'web' service container
docker-compose exec web sh
```

---

## 5. Examples

### Example 1: Simple Web Server

A basic setup to run a Python web server using a custom build.

**Structure:**
```
.
├── docker-compose.yaml
├── app.py
└── Dockerfile
```

**docker-compose.yaml:**
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
```

### Example 2: Full Stack Application (Node.js + Redis + Postgres)

A more complex example showing networking, volumes, and dependencies.

**docker-compose.yaml:**
```yaml
version: '3.8'

services:
  backend:
    image: my-node-app:latest
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - REDIS_HOST=cache
    depends_on:
      - db
      - cache
    networks:
      - app-network

  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    networks:
      - app-network

  cache:
    image: redis:alpine
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```
*In this example, the `backend` service can reach Postgres at hostname `db` and Redis at `cache` because they share the `app-network`.*
