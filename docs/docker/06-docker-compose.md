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

## 2. Docker Compose CLI Commands

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

## 3. Examples

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
