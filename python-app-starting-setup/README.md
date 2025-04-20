# Managing Docker Images and Containers: A Comprehensive Guide

Docker is widely used for containerization, enabling efficient application deployment. This guide covers essential Docker commands and concepts related to managing images and containers, understanding interactive mode, viewing logs, handling file transfers, and optimizing workflows.  

---

## **Getting Help with Docker**  
Docker provides built-in help for commands. To explore available options, simply run:

```sh
docker --help
```
For specific commands, use:

```sh
docker <command> --help
```
This helps in discovering flags and additional functionalities.

---

## **Start vs Run Command: Attached vs Detached Mode**  
When working with containers, understanding how they run is crucial.

### **Docker Run vs Docker Start**
- `docker run <container-id>` → Starts a new container **in attached mode**, meaning the terminal **blocks** while logs are printed.
- `docker start <container-id>` → Starts an **existing container** **in detached mode**, meaning logs won’t be printed, and interaction isn’t possible.

### **Running a Container in Detached Mode**
By default, `docker run` runs **in attached mode**, but you can force it into **detached mode** with:

```sh
docker run -p 8000:80 -d <container-id>
```

Here:
- `-p 8000:80` → Maps container's **port 80** to **host port 8000**.
- `-d` → Runs the container **without blocking the terminal**.

### **Attaching to a Running Container**
Even if a container is running **in detached mode**, you can attach to it anytime:

```sh
docker container attach <container-name>
```

---

## **Viewing Logs for Containers**  

To see logs, use:

```sh
docker logs <container-name>
```

To continuously monitor logs in real-time:

```sh
docker logs -f <container-name>
```

Here, `-f` ensures **live log streaming** in the terminal.

---

## **Interactive Mode in Docker Containers**  
Docker isn't just for web applications—it can also be used for **interactive scripts**.

[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/python-app-starting-setup)

### **Example: Running a Python Script in Interactive Mode**  
Consider a Python script (`rng.py`):

```python
from random import randint

min_number = int(input('Please enter the min number: '))
max_number = int(input('Please enter the max number: '))

if max_number < min_number:
    print('Invalid input - shutting down...')
else:
    rnd_number = randint(min_number, max_number)
    print(rnd_number)
```

#### **Dockerfile for the Python Script**  
```dockerfile
FROM python

WORKDIR /app

COPY . /app

CMD ["python", "rng.py"]
```

#### **Running the Container**
Trying:
```sh
docker run <container-id>
```
Would result in an error.  
Instead, run in **interactive mode**:

```sh
docker run -i -t <container-id>
```
Or equivalently:
```sh
docker run -it <container-id>
```

### **Explanation of `-i` and `-t` Flags**
- `-i` → **Interactive mode**, allowing **input** from the terminal.
- `-t` → **Tty (pseudo-terminal)** mode, enabling **a terminal interface** within the container.

### **Starting an Interactive Container**
By default, `docker start` runs a container **in detached mode**.  
To start in **interactive mode**, use:

```sh
docker start -a -i <container-name>
```
Where:
- `-a` → **Attaches output to the terminal**.
- `-i` → **Allows input interaction**.

---

## **Deleting Containers**  
### **Checking Running Containers**
```sh
docker ps
```

### **Stopping a Running Container**
```sh
docker stop <container-name>
```

### **Removing Containers**
```sh
docker rm <container-name_1> <container-name_2>
```

### **Automatically Removing Stopped Containers**
Instead of manually cleaning up stopped containers, Docker provides the `--rm` flag:

```sh
docker run -p 3000:80 -d --rm <container-id>
```

- `--rm` ensures the container **automatically gets deleted** once it stops.

---

## **Copying Files To and From a Container**  

### **Copying Files INTO a Container**  
```sh
docker cp dummy/. <container-name>:/test
```
This command:
- Copies all files from the local `dummy/` folder **into** `/test` inside the container.
- If `/test` **doesn’t exist**, Docker **automatically creates it**.
- **Use Case**: Copy configuration files **into** a Docker container.

### **Copying Files FROM a Container**  
```sh
docker cp <container-name>:/test /dummy
```
- Copies files **from** `/test` inside the container **to** `dummy/` on the local machine.
- **Use Case**: Extract logs generated inside a container **to the local system**.

---

## **Naming and Tagging Containers & Images**  
### **Naming a Running Container**
When starting a container, you can assign a custom name instead of a random identifier:

```sh
docker run -p 3000:80 --name my-container <image-id>
```
Here:
- `--name my-container` assigns a **human-friendly** name to the container.

### **Naming (Tagging) an Image**
Images can be **tagged** for easier identification:

```sh
docker build -t goals:latest .
```
Where:
- `-t goals:latest` tags the image as **"goals"** with version **"latest"**.

To see all images:
```sh
docker images
```

---

## **Conclusion**
- **Attached vs Detached Mode**: `docker run` is attached by default, `docker start` is detached.
- **Interactive Mode**: `-i` and `-t` allow input/output within a container.
- **Logs Management**: `docker logs -f <container>` provides real-time logs.
- **File Transfers**: Use `docker cp` to move files **to/from a container**.
- **Container Cleanup**: `docker rm` removes stopped containers, `--rm` ensures automatic removal.
- **Naming and Tagging**: Use `--name` for containers and `-t` for images.

By understanding these commands, managing Docker images and containers becomes effortless. Experiment with these concepts in real-world scenarios to strengthen your containerization expertise.  

