## Understanding Kubernetes Objects and Deployment Process

Kubernetes is a powerful container orchestration tool that simplifies the management of application workloads. To fully leverage its capabilities, understanding Kubernetes objects like Pods, Deployments, and Services is essential. This guide provides a technical explanation of Kubernetes dependencies and the process of deploying a simple Node.js application.

### 1. Kubernetes Objects: Pods and Deployments

**1.1 Pods: The Atomic Unit**

In Kubernetes, the smallest deployable unit is a **Pod**. Think of a Pod as a logical grouping of one or more **containers** that are always co-located and co-scheduled. These containers within a Pod share resources such as:

* **Shared Volumes:** Providing persistent storage accessible to all containers within the Pod.
* **Network Namespace:** Allowing containers within the same Pod to communicate via `localhost`.
* **IP Address:** Each Pod is assigned a unique internal IP address within the Kubernetes cluster.

**Analogy:** In the context of AWS Elastic Container Service (ECS), a Pod shares similarities with an ECS **task**.

**Key Characteristic: Ephemeral Nature**

Pods are designed to be **ephemeral**, meaning they are not intended to be long-lived or persistent. Kubernetes can terminate, scale down, or reschedule Pods as needed for various reasons, such as node failures or scaling events. Therefore, directly managing individual Pods is generally not recommended.

**1.2 Deployment Object: Managing Pod Replicas**

To manage the lifecycle and scalability of Pods, Kubernetes utilizes **Controllers**. A crucial controller is the **Deployment** object. Deployments provide declarative updates for Pods and ReplicaSets (a lower-level object that ensures a specified number of Pod replicas are running at any given time).

By defining a **desired state** in a Deployment manifest, you instruct Kubernetes on:

* Which container image(s) to run in the Pods.
* The number of Pod instances (replicas) to maintain.
* The update strategy for rolling out new versions of your application.

**Benefits of using Deployments:**

* **Declarative Configuration:** You define the desired state, and Kubernetes works to achieve and maintain it.
* **Rolling Updates and Rollbacks:** Deployments facilitate seamless application updates with zero downtime and the ability to easily roll back to previous versions if issues arise.
* **Scaling:** You can easily scale the number of Pod replicas up or down based on demand.
* **Self-Healing:** If a Pod fails, the Deployment controller will automatically create a new replica to maintain the desired number of instances.

**In essence, you typically don't directly interact with Pods. Instead, you manage your application's instances through Deployments, which in turn manage the underlying Pods.**

[Practice-Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/kub-action-01-starting-setup)

### 2. Hands-on Deployment of a Node.js Application

Let's walk through the steps to containerize and deploy a simple Node.js application on Kubernetes using Minikube, a tool that runs a single-node Kubernetes cluster locally.

**2.1 Creating a Docker Image**

First, we need to package our Node.js application into a Docker image. The provided `Dockerfile` serves this purpose:

```dockerfile
FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 8080

CMD [ "node", "app.js" ]
```

This `Dockerfile` performs the following actions:

* Starts with a lightweight Alpine Linux-based Node.js version 14 image.
* Sets the working directory inside the container to `/app`.
* Copies the `package.json` file to install dependencies.
* Copies the application code into the `/app` directory.
* Exposes port 8080, which our Node.js application will listen on.
* Specifies the command to run the application when the container starts.

We then build and tag the Docker image:

```bash
docker build -t kub-first-app .
```

To make this image accessible to our Kubernetes cluster, we need to push it to a container registry like Docker Hub.

**2.2 Pushing the Image to Docker Hub**

As mentioned, we create a public repository on Docker Hub (e.g., `mayankcse1/kub-first-app`). After tagging the local image with the repository name:

```bash
docker tag kub-first-app mayankcse1/kub-first-app
```

We push the image to Docker Hub:

```bash
docker push mayankcse1/kub-first-app
```

Now, our container image is publicly available for Kubernetes to pull and run.

**2.3 Starting Minikube**

Before interacting with Kubernetes, we need to ensure our Minikube cluster is running:

```bash
minikube status
```

If it's not running, we can start or restart it using the Docker driver:

```bash
minikube start --driver=docker
```

**2.4 Creating a Kubernetes Deployment**

Now, we can instruct Kubernetes to create a Deployment that will manage Pods running our `mayankcse1/kub-first-app` image:

```bash
kubectl create deployment first-app --image=mayankcse1/kub-first-app
```

**Behind the Scenes of `kubectl create deployment --image`:**

When you execute this command, the `kubectl` command-line tool communicates with the Kubernetes API server. The API server then does the following:

![kubectl-create-command-behind-the-scene](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y4adpb98e0epy9wns3pz.png)

1.  **Creates a Deployment object:** Kubernetes stores the desired state defined in your command (a Deployment named `first-app` running the specified image) in its etcd datastore.
2.  **Triggers the Deployment Controller:** The Deployment controller is a component within the Kubernetes control plane that constantly monitors the state of Deployments. It notices the new Deployment object.
3.  **Creates a ReplicaSet:** Based on the Deployment specification (in this case, the default is one replica), the Deployment controller creates a ReplicaSet object. The ReplicaSet's responsibility is to maintain a stable set of replica Pods running the specified container image.
4.  **Creates Pods:** The ReplicaSet controller then creates the actual Pod(s) based on the Pod template defined in the ReplicaSet. This template includes the container specification (the `mayankcse1/kub-first-app` image).
5.  **Schedules Pods:** The Kubernetes scheduler component determines the most suitable node in the cluster to run the newly created Pod(s) based on resource availability and other constraints.
6.  **Pulls and Runs Containers:** The kubelet agent running on the selected node receives the instruction to run the container(s) defined in the Pod specification. It pulls the `mayankcse1/kub-first-app` image from Docker Hub (or another configured registry) and starts the container(s) using the container runtime (in this case, Docker).

**Verification:**

You can verify the Deployment and the created Pods using the following commands:

```bash
kubectl get deployments
kubectl get pods
```

**Kubernetes Dashboard:**

The Minikube dashboard provides a web-based UI to visualize your Kubernetes cluster:

```bash
minikube dashboard
```

![minikube-dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qaxfgb0ep5dmf4mcfp1d.png)
This dashboard will show you the created Deployment and the Pods it manages.

### 3. Service Object: Exposing Applications

While Pods are running and have internal IP addresses, these IPs are ephemeral and only accessible within the cluster. To make our application accessible within the cluster or externally, we need to use a **Service** object.

A Kubernetes **Service** provides a stable IP address and DNS name to access a set of Pods. It acts as a load balancer and traffic router, ensuring that requests are distributed across the healthy Pods that match the Service's selector.

**Service Types:**

Kubernetes offers different types of Services to expose applications in various ways:

* **`ClusterIP` (Default):** Exposes the Service on a cluster-internal IP. This makes the Service only reachable from within the cluster.
* **`NodePort`:** Exposes the Service on each Node's IP at a static port (the NodePort). You can then access the Service from outside the cluster using the Node's IP and the NodePort. **Drawback:** Only allows one Service to be exposed per node port, and the port range is limited. Also, you still need an external load balancer for high availability in multi-node setups.
* **`LoadBalancer`:** Exposes the Service externally using a cloud provider's load balancer. The cloud provider provisions a load balancer, and external traffic is routed to the NodePort and ClusterIP Service automatically. This type is typically used in cloud environments.

**Exposing our Application:**

For this example, we will use the `LoadBalancer` type (which Minikube simulates):

```bash
kubectl expose deployment first-app --type=LoadBalancer --port=8080
```

This command creates a Service named `first-app` that selects the Pods managed by our `first-app` Deployment and exposes port 8080.

**Verification:**

You can check the created Service using:

```bash
kubectl get services
```

You will see the `first-app` Service listed with an external IP address (or `<pending>` if the load balancer is still being provisioned by Minikube).

**Accessing the Application:**

To access the exposed application, Minikube provides a convenient command:

```bash
minikube service first-app
```

This command will open your default web browser and navigate to the external IP and port where your application is accessible.

![kubernetes-hosted-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2u350979mzi17rzctk7c.png) 

**Conclusion:**

We have successfully deployed a simple Node.js application on Kubernetes using Deployments to manage the application's instances (Pods) and Services to expose it both internally and externally. This demonstrates the fundamental concepts of Pods, Deployments, and Services, which are essential for running and managing containerized applications on Kubernetes. Remember that in real-world scenarios, you would typically define these objects using YAML manifests for better version control and automation.