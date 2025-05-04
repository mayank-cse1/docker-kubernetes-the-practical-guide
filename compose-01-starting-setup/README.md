# Simplifying Multi-Container Development with Docker Compose

In the [last blog post](https://dev.to/mayankcse/building-a-real-world-multi-container-application-using-docker-3eh0), we built a real-world, multi-container application using Docker. However, managing multiple containers manually with long `docker run` commands can become repetitive, error-prone, and hard to maintain—especially in teams or CI/CD environments.

This is where **Docker Compose** comes in. In this post, we’ll explore what Docker Compose is, what it isn’t, how it simplifies development, and how to use it to manage a full stack application with a MongoDB database, Node.js backend, and React frontend.

---

## What is Docker Compose?

**Docker Compose** is a tool designed to help you define and manage multi-container Docker applications. It uses a YAML file (`docker-compose.yml`) to configure application services, networks, volumes, and environment variables. With a single command, you can build, start, and run your entire application stack.

### Benefits of Docker Compose:

* Manage multiple containers as a single service
* Reproducible and consistent development environments
* Simplifies networking and volume management
* Easier to share configurations with teams or deploy to staging

---

## What Docker Compose is **NOT**

It's important to understand what Docker Compose **doesn't** do:

* It does **not** replace Dockerfiles — you still need Dockerfiles to build custom images.
* It does **not** replace Docker containers or images — it simply manages them.
* It is **not** intended for multi-host orchestration (use Kubernetes or Docker Swarm for that).

---

## Installing Docker Compose (Linux)

If you are using macOS or Windows, Docker Compose is included with Docker Desktop.

For Linux:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

More details: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

---

[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/compose-01-starting-setup)

## Defining Our Application in `docker-compose.yml`

We’ll build a basic full-stack application with:

* **MongoDB** database
* **Node.js** backend
* **React** frontend

```yaml
version: "3.8"

services:
  mongodb:
    image: mongo
    container_name: mongodb
    volumes:
      - data:/data/db
    env_file:
      - ./env/mongo.env

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: backend
    ports:
      - '80:80'
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - ./backend/node_modules:/app/node_modules
    env_file: 
      - ./env/backend.env
    depends_on:
      - mongodb

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: frontend
    ports:
      - '3000:3000'
    volumes:
      - ./frontend/src:/app/src
      - ./frontend/node_modules:/app/node_modules
    env_file:
      - ./env/frontend.env
    depends_on:
      - backend
    stdin_open: true
    tty: true

volumes:
  data:
  logs:
```

---

## Breaking Down the Services

### 1. MongoDB Service

Instead of running:

```bash
docker run --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=max \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  -v data:/data/db \
  --rm -d --network goals-net mongo
```

In Compose:

```yaml
mongodb:
  image: mongo
  container_name: mongodb
  volumes:
    - data:/data/db
  env_file:
    - ./env/mongo.env
```

#### Example `mongo.env`:

```
MONGO_INITDB_ROOT_USERNAME=Mayank
MONGO_INITDB_ROOT_PASSWORD=Gupta
```

---

### 2. Backend Service

Instead of manually running build and run commands:

```bash
cd backend
docker build -t goals-node .
docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node
```

In Compose:

```yaml
backend:
  build:
    context: ./backend
    dockerfile: Dockerfile
  container_name: backend
  ports:
    - '80:80'
  volumes:
    - logs:/app/logs
    - ./backend:/app
    - ./backend/node_modules:/app/node_modules
  env_file:
    - ./env/backend.env
  depends_on:
    - mongodb
```

This also supports **live code updates** using bind mounts.

---

### 3. Frontend Service

Manually:

```bash
cd frontend
docker build -t goals-react .
docker run --name goals-frontend --rm -d -p 3000:3000 -it goals-react
```

With Compose:

```yaml
frontend:
  build:
    context: ./frontend
    dockerfile: Dockerfile
  container_name: frontend
  ports:
    - '3000:3000'
  volumes:
    - ./frontend/src:/app/src
    - ./frontend/node_modules:/app/node_modules
  env_file:
    - ./env/frontend.env
  depends_on:
    - backend
  stdin_open: true
  tty: true
```

---

## Running the Application

Start all containers:

```bash
docker-compose up
```

Detached mode:

```bash
docker-compose up -d
```

Stop and remove containers:

```bash
docker-compose down
```

Stop and remove containers and volumes:

```bash
docker-compose down -v
```

Rebuild images before starting:

```bash
docker-compose up --build
```

---

## Advantages of Using Docker Compose

* **One command** to run everything
* Clear **declaration of services and dependencies**
* Easily **version-controlled** with your codebase
* Enables quick **testing** and **CI integration**

---

## Conclusion

Docker Compose helps us streamline multi-container application development by reducing complexity and enabling declarative service definitions. It's ideal for local development, prototyping, and small-scale deployments.

In upcoming blogs, we’ll cover how to use Docker Compose for CI/CD, working with environment-specific files, and Docker Compose for production scenarios.

Explore the source and more projects on my [Dev.to profile](https://dev.to/mayankcse). If you haven’t checked the first part yet, read it [here](https://dev.to/mayankcse/building-a-real-world-multi-container-application-using-docker-3eh0).
