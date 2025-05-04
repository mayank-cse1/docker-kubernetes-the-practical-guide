# Building a Real-World Multi-Container Application Using Docker

In previous posts, we explored Docker basics, container networking, and AI integrations. If you haven't checked them out yet, visit [my blog on Dev.to](https://dev.to/mayankcse).

In this blog, we'll combine our Docker learnings to build a more sensible, real-world application using multiple containers. A typical full-stack application is composed of three essential building blocks:

**Database ⇄ Backend ⇄ Frontend**

We'll walk through building a multi-container setup for a course goals application using **MongoDB**, **Node.js**, and **React**.

---

## Prerequisites

* Docker installed and running
* Basic understanding of Node.js, React, and MongoDB
* Familiarity with Dockerfile, volumes, and networks

---
[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/multi-01-starting-setup)

## Step 1: Dockerizing MongoDB (Database)

MongoDB offers a ready-to-use Docker image. Start a MongoDB container with:

```bash
docker run --name mongodb --rm -d -p 27017:27017 mongo
```

### Explanation:

* `-p 27017:27017`: Maps the MongoDB port to localhost
* `-d`: Runs in detached mode
* `--rm`: Automatically removes the container when stopped

However, if you try to connect from a containerized backend using:

```js
mongoose.connect('mongodb://localhost:27017/course-goals')
```

It will **fail**, because `localhost` in a container refers to the container itself. To solve this, change it to:

```js
mongoose.connect('mongodb://host.docker.internal:27017/course-goals')
```

This enables access to the host system from within a Docker container.

---

## Step 2: Dockerizing the Backend (Node.js + Express)

Navigate to the `backend/` folder and create a `Dockerfile`:

```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 80
CMD ["node", "app.js"]
```

Build and run the backend container:

```bash
docker build -t goals-node .
docker run --name goals-backend --rm -p 80:80 -d goals-node
```

---

## Step 3: Dockerizing the Frontend (React)

Navigate to the `frontend/` folder and create a `Dockerfile`:

```dockerfile
FROM node
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

Build and run the frontend:

```bash
docker build -t goals-react .
docker run --name goals-frontend --rm -d -it -p 3000:3000 goals-react
```

Now, your app is accessible via `http://localhost:3000`.

---

## Step 4: Creating a Docker Network

Let’s create a Docker network so the containers can communicate internally:

```bash
docker network create goals-net
```

Re-run MongoDB inside this network:

```bash
docker run --name mongodb --rm -d --network goals-net mongo
```

Update backend connection:

```js
mongoose.connect('mongodb://mongodb:27017/course-goals')
```

The container name `mongodb` acts like a hostname here.

Rebuild and run the backend:

```bash
docker build -t goals-node .
docker run --name goals-backend --rm -d -p 80:80 --network goals-net goals-node
```

Note: **Frontend (browser-based) apps cannot use container names to make HTTP requests**. Hence, exposing backend on the host (localhost) using `-p 80:80` is still required.

---

## Step 5: Adding Data Persistence to MongoDB

Without persistence, MongoDB data is lost when the container stops. Create a named volume:

```bash
docker run --name mongodb -v data:/data/db --rm -d --network goals-net mongo
```

Now the data survives restarts.

---

## Step 6: Enabling Authentication in MongoDB

Mongo supports setting root user credentials using environment variables:

```bash
docker run --name mongodb -v data:/data/db --rm -d --network goals-net \
-e MONGO_INITDB_ROOT_USERNAME=Mayank \
-e MONGO_INITDB_ROOT_PASSWORD=Gupta \
mongo
```

Update the backend connection string:

```js
mongoose.connect(
  'mongodb://Mayank:Gupta@mongodb:27017/course-goals?authSource=admin'
)
```

---

## Step 7: Live Source Code Updates with Bind Mounts

Bind mounts are useful in development to reflect live code changes without rebuilding.

Example backend run command:

```bash
docker run --name goals-backend \
-v $(pwd):/app \
-v logs:/app/logs \
-v /app/node_modules \
--rm -p 80:80 \
--network goals-net \
goals-node
```

Frontend with live code updates:

```bash
docker run -v $(pwd)/frontend/src:/app/src \
--name goals-frontend --rm -p 3000:3000 -it goals-react
```

---

## Step 8: Using Environment Variables

Avoid hardcoding credentials by using ENV variables.

**Update Dockerfile:**

```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 80
ENV MONGODB_USERNAME=Mayank
ENV MONGODB_PASSWORD=Gupta
CMD ["node", "app.js"]
```

**Update connection string in code:**

```js
const uri = `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`
mongoose.connect(uri)
```

**Run backend container:**

```bash
docker run --name goals-backend \
-v $(pwd):/app \
-v logs:/app/logs \
-v /app/node_modules \
--rm -p 80:80 \
-e MONGODB_USERNAME=Mayank \
-e MONGODB_PASSWORD=Gupta \
--network goals-net \
goals-node
```

---

## Step 9: Optimizing with .dockerignore

Create a `.dockerignore` file in both frontend and backend directories:

```
node_modules
.git
Dockerfile
```

This reduces the build context and speeds up Docker builds.

---

## Conclusion

You now have a production-ready multi-container application using Docker for:

* MongoDB (database)
* Node.js (backend API)
* React (frontend UI)

We leveraged Docker volumes, environment variables, bind mounts, authentication, and inter-container networking — all essential for real-world applications.

This practice not only prepares you for scalable deployments but also mirrors the infrastructure used in many modern microservices architectures.


