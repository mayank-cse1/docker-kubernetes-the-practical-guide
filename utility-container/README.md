# Utility Containers in Docker: A Powerful Tool for Local Development Without Polluting Your Machine

In our previous blogs, we explored how to build real-world multi-container applications using Docker and simplify container orchestration using Docker Compose. In this post, we’ll delve into **utility containers**, an often-overlooked but extremely powerful use of Docker that can dramatically streamline your development workflow.

## Why Utility Containers?

Imagine you're starting a fresh Node.js project. Typically, you’d run:

```bash
npm init
```

But to run that, you'd need Node.js installed on your host machine. What if you're working on a lightweight Linux server, a CI/CD pipeline, or a constrained environment where you **don’t want to install Node.js globally**?

This is where utility containers come in. They allow you to run one-off or repetitive tasks using Docker containers without installing dependencies on your local machine.

---

## Step 1: Create a Utility Container

Let’s create a Dockerfile that uses Node.js as its base:

**Dockerfile**

```Dockerfile
FROM node:14-alpine
WORKDIR /app
```

We're using the official lightweight Node.js image. The `WORKDIR` directive sets the default working directory inside the container to `/app`.

We **don’t add a CMD** because we want this container to be flexible – capable of running any npm command like `npm init`, `npm install`, or `npm test`.

Now build the Docker image:

```bash
docker build -t node-util .
```

---

## Step 2: Run the Utility Container

Let’s say you have a project directory at `/Users/mayank/projects/util-node`. Navigate to it and run:

```bash
docker run -it -v $(pwd):/app node-util npm init
```

* `-it`: Runs the container interactively
* `-v $(pwd):/app`: Mounts your current directory to the container’s `/app` directory
* `node-util`: The image you built
* `npm init`: The command to execute inside the container

After the command finishes, you’ll notice a `package.json` file generated in your local folder.

---

## Step 3: Simplify with ENTRYPOINT

Let’s make it even easier. Add an ENTRYPOINT instruction to the Dockerfile so you don’t have to type `npm` every time:

**Updated Dockerfile**

```Dockerfile
FROM node:14-alpine
WORKDIR /app
ENTRYPOINT ["npm"]
```

Now you can run commands like this:

```bash
docker run -it -v $(pwd):/app node-util init
```

or

```bash
docker run -it -v $(pwd):/app node-util install express
```

Docker will automatically prepend `npm` to your command. This is extremely helpful in reducing errors and typing effort.

---

## Step 4: Use Docker Compose for Even More Simplicity

Running the long `docker run` command repeatedly can be tiring. Let’s use Docker Compose to define the setup once.

**docker-compose.yaml**

```yaml
version: '3.8'
services:
  npm:
    build: ./
    stdin_open: true
    tty: true
    volumes:
      - ./:/app
```

Now you can run:

```bash
docker-compose run npm init
```

or

```bash
docker-compose run npm install express
```

Docker Compose takes care of mounting the volume and setting up the working directory, so you don’t have to repeat those parameters every time.

---

## Bonus: Using Utility Containers for Other Tools

You’re not limited to Node.js. You can create utility containers for:

* **Python**: `FROM python:3.10`
* **Go**: `FROM golang:1.20`
* **Terraform**: `FROM hashicorp/terraform:latest`

Just change the base image and entrypoint accordingly.

---

## Conclusion

Utility containers are a powerful and clean way to run development tasks without polluting your host machine. When combined with Docker Compose, they offer a repeatable, team-friendly development experience. They are ideal for bootstrapping projects, running tests, building apps, or using tools that aren’t worth installing globally.

Instead of installing language runtimes or package managers on every machine, create utility containers once and use them everywhere.

---
