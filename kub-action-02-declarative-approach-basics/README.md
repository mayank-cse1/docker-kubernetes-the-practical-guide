In the last blog, we explored the basics of Kubernetes. Just like Docker, where lengthy `docker run` commands were simplified with `docker-compose`, Kubernetes offers a similar improvement through resource definition files.

## Imperative vs Declarative Approach

### **Imperative Approach**
- You manually execute commands to trigger Kubernetes actions.
- Example:  
  ```sh
  kubectl create deployment <name> --image=<image>
  ```
- Comparable to using `docker run` for container management.

### **Declarative Approach**
- You define a desired state in a YAML configuration file and apply it.
- Example:
  ```sh
  kubectl apply -f config.yaml
  ```
- Comparable to using `docker-compose` with a compose file.

The declarative approach simplifies management, reduces errors, and allows for easy sharing.

---
[Practice Resource](https://github.com/mayank-cse1/docker-kubernetes-the-practical-guide/tree/main/kub-action-02-declarative-approach-basics)

## **Creating a Deployment Configuration File (Declarative Approach)**

### **1. Start the Minikube cluster**
```sh
minikube start
```

### **2. Check existing deployments**
```sh
kubectl get deployments
kubectl get pods
kubectl get services
```
If there are existing deployments, delete them using:
```sh
kubectl delete deployments <name>
kubectl delete pods <name>
kubectl delete services <name>
```

---

### **Deployment Configuration (deployment.yaml)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: second-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: second-app
      tier: backend
  template:
    metadata: 
      labels:
        app: second-app
        tier: backend
    spec: 
      containers:
        - name: second-node
          image: academind/kub-first-app:2
```
#### **Explanation**
- **`apiVersion`**: Specifies the Kubernetes API version being used (`apps/v1` for Deployment).
- **`kind`**: Defines the object type (`Deployment`).
- **`metadata`**: Contains the name of the deployment (`second-app-deployment`).
- **`spec`**: Specifies the desired state of the deployment.
  - **`replicas`**: Defines how many instances should be running (`1`).
  - **`selector`**: Ensures the deployment only manages pods with labels `app: second-app` and `tier: backend`.
  - **`template`**: Defines the pod specification.
    - **`labels`**: Matches the labels used in `selector`.
    - **`containers`**: Specifies the container name (`second-node`) and its image (`academind/kub-first-app:2`).

### **Apply the Deployment**
```sh
kubectl apply -f deployment.yaml
```

### **Verify Deployment**
```sh
kubectl get deployments
kubectl get pods
```

---

### **Creating a Kubernetes Service (service.yaml)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector: 
    app: second-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

#### **Explanation**
- **`kind`**: Defines that this is a `Service`.
- **`metadata`**: Specifies the service name (`backend`).
- **`selector`**: Associates the service with pods having label `app: second-app`.
- **`ports`**: Maps incoming traffic on port `80` to container port `8080`.
- **`type`**:
  - `LoadBalancer`: Exposes the service externally using a cloud provider’s load balancer.
  - `NodePort`: Exposes the service on each Node’s IP at a static port.
  - `ClusterIP`: Restricts access within the cluster.

### **Apply the Service**
```sh
kubectl apply -f service.yaml
```

### **Access the Service**
```sh
minikube service backend
```

![kubernetes-deployment-success](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/h8mqp2n0u9gmgx4rkr5q.png)
---

### **Why Declarative Approach is Better**
- It is more **structured**, **easily shared**, and **less error-prone**.
- You don't need long commands for modifications—just edit the YAML file and reapply.
- Updating existing objects is straightforward:
  ```sh
  kubectl apply -f deployment.yaml
  ```

### **Deleting Resources**
Deletion can be done using either approach:

#### **Imperative Approach**
```sh
kubectl delete deployments <name>
kubectl delete services <name>
```

#### **Declarative Approach**
```sh
kubectl delete -f deployment.yaml -f service.yaml
```

---

### **Merging Deployment and Service into a Single YAML File**
Instead of maintaining separate YAML files, merge them using triple hyphens (`---`):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector: 
    app: second-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer 

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: second-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: second-app
      tier: backend
  template:
    metadata: 
      labels:
        app: second-app
        tier: backend
    spec: 
      containers:
        - name: second-node
          image: mayankcse1/kub-first-app:latest
```

### **Why Service Comes First?**
Since YAML files are processed **top-down**, the **Service** should be created first so objects can be tagged to it.

---

### **Advanced Label Selection with `matchExpressions`**
Instead of `matchLabels`, you can use `matchExpressions` for advanced filtering:

```yaml
selector:
  matchExpressions:
    - {key: app, operator: In, values: [second-app]}
    - {key: tier, operator: In, values: [backend]}
```

---

## **Conclusion**
Using the **declarative approach** in Kubernetes offers a more reliable, scalable, and maintainable way to manage applications. It enables version control, reduces human error, and simplifies the update process.

By defining everything in configuration files, Kubernetes ensures consistency and automation, making infrastructure management seamless.