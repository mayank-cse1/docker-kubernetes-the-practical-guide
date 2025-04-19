# Node.js Application with Docker

## Introduction
Instead of manually installing Node.js dependencies using `npm install`, we use Docker to build and run our application as a container, simplifying setup and deployment.

---

## **Docker Setup**

### **Dockerfile**
The following **Dockerfile** defines how our Node.js application is built inside a container:

```dockerfile
FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD [ "node", "app.mjs" ]
```
This setup:
- Uses `node:14` as the base image.
- Sets `/app` as the working directory.
- Copies `package.json` and runs `npm install` inside the container.
- Copies the rest of the application files.
- Exposes **port 3000**.
- Runs `app.mjs` using Node.js.

---

## **Building the Docker Image**
Navigate to the project folder:

```sh
cd FIRST-DEMO-STARTING-SETUP
```

Run the following command to **build the image**:

```sh
docker build .
```

### **Troubleshooting**
- If you encounter any errors, restart **Docker Desktop** and try again.

After a successful build, you will see an image ID at the last line:

```
=> unpacking to sau-dang@sha256:8b46416a25b73c90ae3262530cb2435d5cbc22020aaeeaade8ET3455R9ret23f1er34a9b2
```

This **image ID** will be used to run the container.

---

## **Running the Container**
Use the **image ID** to start the container:

```sh
docker run -p 3000:3000 <docker-id>
```

Replace `<docker-id>` with the actual image ID from the build output.

---
<img width="940" alt="Docker Success" src="https://github.com/user-attachments/assets/8cbd9050-e116-4c80-b358-bad43471196e" />

## **Viewing Running Containers**
To see all active containers:

```sh
docker ps
```

Example output:

```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                    NAMES
41934adffdba   8b46416a25b7   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:3000->3000/tcp   elegant_lewin
```

---

## **Stopping the Container**
To stop a running container:

```sh 
docker stop elegant_lewin
```

Replace `elegant_lewin` with your container name from the `docker ps` output.

---

### **Conclusion**
Using Docker simplifies the setup and deployment of our Node.js application. It eliminates the need for manual package installations and ensures consistency across environments.

---
