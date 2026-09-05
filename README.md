# 🚀 ZIDD 3.0 by Devops_jadeja - Docker Workshop

Welcome to the **ZIDD 3.0 Employee Portal & DevOps Dashboard** project! This repository serves as a hands-on application designed for the **Docker Workshop**, demonstrating local Node.js application management, containerization with Docker, multi-container orchestration with Docker Compose, and essential Docker concepts including **Volumes** and **Networks**.

---

## 📋 Table of Contents
1. [Project Overview](#-project-overview)
2. [Running Locally (Without Docker)](#-running-locally-without-docker)
3. [Containerizing with Docker](#-containerizing-with-docker)
4. [Docker Concepts & CLI Guide](#-docker-concepts--cli-guide)
   - [1. Docker Images & Containers](#1-docker-images--containers)
   - [2. Docker Volumes (Data Persistence)](#2-docker-volumes-data-persistence)
   - [3. Docker Networks (Container Communication)](#3-docker-networks-container-communication)
5. [Multi-Container Setup with Docker Compose](#-multi-container-setup-with-docker-compose)
6. [Quick Command Reference](#-quick-command-reference)

---

## ℹ️ Project Overview

- **Stack**: Node.js, Express, HTML5, Vanilla CSS / Tailwind CSS
- **Port**: `3000`
- **Author**: `Devops_jadeja`
- **Workshop Edition**: `ZIDD 3.0`

The server exposes key dashboard endpoints:
- `GET /` - Renders the **ZIDD 3.0 Docker Workshop** frontend dashboard.
- `GET /api/dashboard` - Returns system health, container status, and request counts.
- `GET /api/health` - Basic application health probe (`HTTP 200`).

---

## 💻 Running Locally (Without Docker)

### Prerequisites
- Node.js (v18 or higher)
- npm (Node Package Manager)

### Step-by-Step Instructions

1. **Install Dependencies**:
   ```bash
   npm install
   ```

2. **Start the Server**:
   ```bash
   npm start
   # Or directly:
   node server.js
   ```

3. **Access the Application**:
   Open your browser and navigate to:
   [http://localhost:3000](http://localhost:3000)

---

## 🐳 Containerizing with Docker

The repository includes a production-ready `Dockerfile`:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

### 1. Build the Docker Image
To build the Docker image for this application, run:
```bash
docker build -t zidd-employee-portal:v1.0 .
```

### 2. Run the Docker Container
Run the container in detached mode (`-d`), mapping port `3000` on your host machine to port `3000` inside the container:
```bash
docker run -d -p 3000:3000 --name zidd-portal-container zidd-employee-portal:v1.0
```

### 3. Verify Container Execution
Open your browser at [http://localhost:3000](http://localhost:3000) or run:
```bash
curl http://localhost:3000/api/health
```

### 4. Stop and Remove Container
```bash
# Stop the running container
docker stop zidd-portal-container

# Remove the container
docker rm zidd-portal-container
```

---

## 📘 Docker Concepts & CLI Guide

### 1. Docker Images & Containers

- **Image**: A read-only template containing application code, runtime, libraries, and environment variables.
- **Container**: A runnable instance of a Docker image.

```bash
# List all downloaded/built images
docker images

# List active running containers
docker ps

# List all containers (including stopped ones)
docker ps -a

# View real-time container logs
docker logs -f zidd-portal-container

# Execute an interactive shell inside a running container
docker exec -it zidd-portal-container sh
```

---

### 2. Docker Volumes (Data Persistence)

Docker containers are ephemeral—data written inside a container disappears when the container is deleted. **Docker Volumes** persist data outside the container lifecycle on the host machine.

#### Types of Storage Mounts:
1. **Named Volumes**: Managed completely by Docker in `/var/lib/docker/volumes/`.
2. **Bind Mounts**: Mounts a specific host directory directly into the container.

#### Volume Commands:
```bash
# Create a named volume
docker volume create zidd-data-vol

# List all volumes
docker volume ls

# Inspect volume details (shows host mount path)
docker volume inspect zidd-data-vol

# Remove a volume
docker volume rm zidd-data-vol

# Remove all unused volumes
docker volume prune
```

#### Running Container with a Volume:
```bash
# 1. Named Volume Mount:
docker run -d -p 3000:3000 \
  -v zidd-data-vol:/app/data \
  --name zidd-app \
  zidd-employee-portal:v1.0

# 2. Bind Mount (mount current host directory into /app inside container for live development):
docker run -d -p 3000:3000 \
  -v $(pwd):/app \
  --name zidd-app-dev \
  zidd-employee-portal:v1.0
```

---

### 3. Docker Networks (Container Communication)

Docker Networks allow containers to communicate with each other securely using container names as DNS hostnames.

#### Network Drivers:
- `bridge` *(Default)*: Private internal network on the host.
- `host`: Shares the host machine's network stack directly.
- `none`: Disables networking completely for isolation.

#### Network Commands:
```bash
# Create a custom bridge network
docker network create mynet

# List all networks
docker network ls

# Inspect network and connected containers
docker network inspect mynet

# Connect a running container to a network
docker network connect mynet zidd-portal-container

# Disconnect container from network
docker network disconnect mynet zidd-portal-container

# Remove network
docker network rm mynet
```

#### Example: Running Application & Database on the Same Network
```bash
# 1. Create custom network
docker network create workshop-net

# 2. Run MySQL Database container on workshop-net
docker run -d \
  --name mysql-db \
  --network workshop-net \
  -e MYSQL_ROOT_PASSWORD=secret123 \
  mysql:8.0

# 3. Run Node.js Application container on workshop-net
docker run -d \
  --name zidd-app \
  --network workshop-net \
  -p 3000:3000 \
  zidd-employee-portal:v1.0
```

---

## 🐙 Multi-Container Setup with Docker Compose

`docker-compose.yaml` allows you to define and manage multi-container applications (Web + Database + Networks + Volumes) declaratively.

### Compose File Overview (`docker-compose.yaml`)
```yaml
version: '2.2'

services:
  web:
    image: nginx:latest
    container_name: nginx-web
    ports:
      - "8080:80"
    networks:
      - mynet

  db:
    image: mysql:8.0
    container_name: mysql-db
    environment:
      MYSQL_ROOT_PASSWORD: secret123
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - mynet

networks:
  mynet:

volumes:
  dbdata:
```

### Docker Compose Commands

```bash
# Start all containers in background (-d)
docker compose up -d

# Check status of stack containers
docker compose ps

# View aggregate logs for all services
docker compose logs -f

# View logs for a specific service
docker compose logs -f web

# Stop all running containers without destroying volumes
docker compose stop

# Tear down containers, networks, and shutdown stack
docker compose down

# Tear down containers, networks, AND volumes (removes persisted data)
docker compose down -v
```

---

## ⚡ Quick Command Reference

| Task | Command |
| :--- | :--- |
| **Run Locally** | `npm install && node server.js` |
| **Build Image** | `docker build -t zidd-employee-portal:v1.0 .` |
| **Run Container** | `docker run -d -p 3000:3000 --name app zidd-employee-portal:v1.0` |
| **View Logs** | `docker logs -f app` |
| **Shell Access** | `docker exec -it app sh` |
| **Create Volume** | `docker volume create my-vol` |
| **Create Network**| `docker network create my-net` |
| **Start Compose** | `docker compose up -d` |
| **Stop Compose**  | `docker compose down` |

---

<p center="text-center">
<strong>ZIDD 3.0 by Devops_jadeja • Docker Workshop Guide</strong>
</p>
# docker-zidd3
