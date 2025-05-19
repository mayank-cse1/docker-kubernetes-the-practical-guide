## MANAGING DATA & VOLUMES IN KUBERNETES

Imagine you’re running an application that lets users store text, but every time the container crashes or restarts, their data is lost. Frustrating, right? That’s exactly the problem Kubernetes solves with volumes—ensuring data survives beyond container lifecycles. In this blog, we’ll walk through a practical example that demonstrates how Kubernetes can persist data effectively, even in failure scenarios.

## **The Problem: Where Did My Data Go?**

Containers are lightweight, flexible, and easily restartable, but they come with a challenge: they are **stateless by default**. This means that every time a container crashes or is redeployed, any data stored inside it vanishes. 

[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/kub-data-01-starting-setup)

Consider a simple application where users submit text, and the text is stored in a file inside a container. Here's how you can run it using Docker Compose:

```sh
docker compose up --build
```

Once the container is running, test it using `curl`:

- Retrieve stored text:
  ```sh
  curl --location 'localhost/story'
  ```

- Add new text:
  ```sh
  curl --location 'localhost/story' \
    --header 'Content-Type: application/json' \
    --data '{
        "text": "My text!"
    }'
  ```

Now, if the container crashes and restarts, all previously submitted text is gone! This happens because the container filesystem resets on every startup.

So, how do we **persist** the data beyond container failures? Kubernetes volumes provide the answer.

## **Solution: Adding Volumes in Kubernetes**

Kubernetes allows containers to mount **volumes**—storage spaces that persist beyond container lifecycles. Compared to Docker volumes, Kubernetes volumes offer greater flexibility and resilience.

### **Setting Up Kubernetes Deployment**

First, push the Docker image to a registry:

```sh
docker tag kub-data-01-starting-setup-stories mayankcse1/kub-data-01-starting-set
docker push mayankcse1/kub-data-01-starting-setup-stories
```

Then, create a `deployment.yaml` file to define how Kubernetes should manage our container:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: story
  template:
    metadata:
      labels:
        app: story
    spec:
      containers:
        - name: story
          image: mayankcse1/kub-data-01-starting-setup-stories
          ports:
            - containerPort: 3000
          volumeMounts:
            - mountPath: /app/story
              name: stories-volume
      volumes:
        - name: stories-volume
          persistentVolumeClaim:
            claimName: stories-pvc
```

### **Creating a Service to Expose the App**

Now, define a Kubernetes service to allow external access to the application:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: story-service
spec:
  selector:
    app: story
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

## **Deploy and Test the Application in Kubernetes**

Start Minikube if it's not running:

```sh
minikube status
minikube start --driver=docker
```

Then, apply the Kubernetes configuration:

```sh
kubectl apply -f=service.yaml -f=deployment.yaml
kubectl get pods
kubectl get deployments
minikube service story-service
```

Test the service:

```sh
curl --location 'http://<Host Address>/story'
curl --location 'http://<Host Address>/story' \
--header 'Content-Type: application/json' \
--data '{
    "text": "mayank"
}'
```

At this point, the service is running—but our data can still **disappear** when the pod itself is removed. To solve this, let’s improve our volume strategy.

## **Ensuring Data Persistence with Volumes**

Kubernetes supports various volume types, but a simple way to persist data within a pod is using `emptyDir`:

```yaml
volumes:
  - name: story-volume
    emptyDir: {}
```

This ensures data remains available as long as the **pod** is alive. However, if the pod is **deleted**, all data is lost.

## **Handling Multiple Pods & Node Failures**

If you scale your application to multiple pods, data consistency becomes tricky. Suppose a pod crashes and traffic is redirected to another pod—the new pod won’t have the old data!

To store data across multiple pods running on the same **node**, we can use `hostPath`:

```yaml
volumes:
  - name: story-volume
    hostPath:
      path: /data
      type: DirectoryOrCreate
```

However, **this only works for pods on the same node**—if a pod is scheduled on a different node, it won’t have access to the previous data.

For a more robust solution across multiple nodes, consider **Persistent Volumes (PVs)** that work with cloud storage or external databases.

## **Final Thoughts: Why Kubernetes Volumes Matter**

Kubernetes volumes solve **critical data loss issues** in containerized applications. By implementing persistent storage solutions, developers ensure that user data survives container crashes, pod restarts, and even scaling across multiple nodes.

To explore all available volume storage options in Kubernetes, visit the official documentation:  
[https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/)  

Understanding how **stateful applications** work within Kubernetes is essential for building scalable, resilient infrastructure. Whether deploying a simple text storage app or a large-scale distributed system, **managing volumes effectively ensures reliable data persistence**.
