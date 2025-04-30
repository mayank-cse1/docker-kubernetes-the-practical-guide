In modern application development, **Docker** plays a crucial role by allowing services to be isolated into individual containers. However, these services often need to **communicate** with each other or with the **external world (Internet)**. In this blog, we will explore container-to-external communication, container-to-local machine communication, and cross-container communication using **networks** — all with practical examples.

[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/networks-starting-setup)

## A Practice Application

We will use a simple **Node.js** application that has the following dependencies:

- `axios`
- `express`
- `body-parser`
- `mongoose`

The app exposes **4 endpoints**:
- `GET /favorites` — Fetch favorite movies from local MongoDB
- `POST /favorites` — Add a favorite movie to local MongoDB
- `GET /movies` — Fetch movie data from an external third-party API
- `GET /people` — Fetch people data from an external third-party API

External API reference:  
- [Star Wars API - Postman](https://www.postman.com/dragon87/exploration-swapi-api/documentation/2bqv73u/star-wars-api-swapi)  
- [Star Wars API - Official](https://swapi.dev/api/)

---

## Step 1: Building and Running the Container

First, **build the Docker image**:

```bash
docker build -t favorites-node .
```

- `docker build`: Command to build the image.
- `-t favorites-node`: Tagging the image with the name `favorites-node`.
- `.`: Context is the current directory.

Next, **run the container**:

```bash
docker run --name favorites --rm -p 3000:3000 favorites-node
```

- `--name favorites`: Give a name to the container.
- `--rm`: Automatically remove the container after it exits.
- `-p 3000:3000`: Map port 3000 of the container to port 3000 of the local machine.
- `favorites-node`: The image to run.

---

## Step 2: Issue With Local MongoDB

When the application tries to connect to the local MongoDB:

```javascript
mongoose.connect(
  'mongodb://localhost:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => { /* callback */ }
);
```

It **fails** because **localhost inside the container refers to the container itself**, not your local machine.

If you comment out the MongoDB connection code and run the container, the `/movies` and `/people` endpoints work because they connect to the external world without any issues. This proves **containers can connect to the Internet by default**.

---

## Step 3: Connect to Local Machine From Container

To fix the MongoDB connection issue, update the connection string to:

```javascript
mongoose.connect(
  'mongodb://host.docker.internal:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => { /* callback */ }
);
```

Here, `host.docker.internal` points to your local machine from inside the Docker container.

Now running the container again will successfully connect the Node.js app inside the container to MongoDB running on your local machine.

---

## Step 4: Running MongoDB as a Container

Instead of relying on a local MongoDB server, let's run MongoDB in its own container.

```bash
docker run -d --name mongodb mongo
```

- `-d`: Run in detached mode (in the background).
- `--name mongodb`: Name the container.
- `mongo`: Use the official MongoDB Docker image.

Now inspect the MongoDB container:

```bash
docker container inspect mongodb
```

Look for the `IPAddress` field. Suppose it is `172.17.0.2`.

Update the Node.js connection:

```javascript
mongoose.connect(
  'mongodb://172.17.0.2:27017/swfavorites',
  { useNewUrlParser: true },
  (err) => { /* callback */ }
);
```

This works because now the app container can reach the MongoDB container via its IP.

---

## Step 5: Simplifying With Docker Networks

Manually managing IPs is tedious. Instead, we can **create a Docker network**.

### Steps:

1. **Stop running containers**:

   ```bash
   docker stop favorites
   docker stop mongodb
   docker container prune
   ```

2. **Create a custom network**:

   ```bash
   docker network create favorites-net
   ```

3. **Run MongoDB in the new network**:

   ```bash
   docker run -d --name mongodb --network favorites-net mongo
   ```

4. **Update Node.js connection string** to use container name:

   ```javascript
   mongoose.connect(
     'mongodb://mongodb:27017/swfavorites',
     { useNewUrlParser: true },
     (err) => { /* callback */ }
   );
   ```

5. **Build and run the Node.js container in the same network**:

   ```bash
   docker run --name favorites --network favorites-net -d --rm -p 3000:3000 favorites-node
   ```

Now, the app can communicate with MongoDB simply using the container name (`mongodb`), no need to worry about IP addresses.

---

## How Docker Handles This Behind the Scenes

When a container tries to connect using another container's name inside the same network, **Docker automatically resolves the name to the right IP address**.  
No changes to your source code are necessary beyond using the correct container name.

Docker acts like a DNS resolver within the network.

---

## Bonus: Docker Network Drivers

Docker supports different types of **network drivers**:

| Driver | Behavior |
| :--- | :--- |
| **bridge** (default) | Standard network where containers can communicate if in the same network. |
| **host** | Removes network isolation between container and host. |
| **overlay** | Allows containers on different machines to communicate (requires Swarm mode). |
| **macvlan** | Assigns a MAC address to a container for direct communication on the network. |
| **none** | Disables networking completely. |
| **Third-party plugins** | Add custom behavior and advanced networking features. |

To specify a driver while creating a network:

```bash
docker network create --driver bridge my-net
```

The `bridge` driver is the most common and sufficient for most single-machine development and deployment scenarios.

---

# Summary

We learned:
- Containers can communicate with the Internet easily.
- Containers cannot reach your local machine using `localhost` but can use `host.docker.internal`.
- Container-to-container communication works via IPs but is better managed using a **Docker network**.
- Docker networks automatically resolve container names to IPs.
- Different network drivers offer different capabilities for your needs.

Mastering container networking is key to building production-ready distributed applications!
