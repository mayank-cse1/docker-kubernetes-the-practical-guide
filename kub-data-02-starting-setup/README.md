## From Volume to Persistent Volume in Kubernetes

In our [previous blog](https://dev.to/mayankcse/managing-data-volumes-in-kubernetes-1d93), we explored how Kubernetes volumes help preserve data across container restarts. We worked with `emptyDir`, which retains data as long as the **pod** is running but loses it when the pod is deleted. Then, we improved this setup using `hostPath`, which allows a container to persist data at a specified directory on the node.

This worked seamlessly in **Minikube** because it runs on a **single-node cluster**. But what happens in **multi-node cloud environments**? The same solution will fail because Kubernetes dynamically schedules pods across multiple nodes, meaning the data stored in `hostPath` on one node **won’t be available to a pod running on another node**.

To solve this, we need **Persistent Volumes (PVs)** and **CSI (Container Storage Interface)**. 

## **Introducing Persistent Volumes**

A **Persistent Volume (PV)** is independent of individual pods and nodes, providing a stable and reusable storage location across the cluster. 

To configure it, we create a new file `host-pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Block
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
    type: DirectoryOrCreate
```

### **Breaking Down the Configuration**

1. **`apiVersion: v1` & `kind: PersistentVolume`**  
   Defines the resource type as a Persistent Volume.

2. **`metadata.name: host-pv`**  
   Assigns a unique name to the PV, making it identifiable when claimed.

3. **Storage Capacity (`capacity.storage: 1Gi`)**  
   Specifies the storage capacity, set to **1GiB** in this example.

4. **Volume Mode (`volumeMode: Block`)**  
   Declares the volume mode. `Block` is useful for low-level storage needs, but many use `Filesystem` for general applications.

5. **Access Modes (`accessModes`)**  
   Controls how pods can interact with the volume:
   - `ReadWriteOnce`: Only one pod can mount it as read-write.
   - `ReadOnlyMany`: Multiple pods can access it, but only in read-only mode.
   - `ReadWriteMany`: Multiple pods can read and write simultaneously.

6. **Host Path (`hostPath`)**  
   Maps `/data` on the node as the volume’s storage location. `type: DirectoryOrCreate` ensures the directory exists.

## **Claiming the Persistent Volume**

A **Persistent Volume Claim (PVC)** requests a PV from Kubernetes and ensures a pod can access it. Define this in `host-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-pvc
spec:
  volumeName: host-pv
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
```

### **Explanation**
1. **`volumeName: host-pv`**  
   Directly references our previously created PV.

2. **Access Modes (`accessModes`)**  
   Specifies access rights, ensuring only one pod can write at a time.

3. **Storage Class (`storageClassName: standard`)**  
   Defines the underlying storage provisioner. If using cloud services like AWS or GCP, this would be different (e.g., `gp2` for AWS).

4. **Resource Requests (`resources.requests.storage: 1Gi`)**  
   Requests **1GiB** of storage from an available PV.

## **Integrating Persistent Volume into Kubernetes Deployment**

Finally, update `deployment.yaml` to use the **PVC**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: story-deployment
spec:
  replicas: 2
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
          image: mayankcse1/kub-data-01-starting-setup-stories:1
          volumeMounts:
            - name: story-volume
              mountPath: /app/story
      volumes:
        - name: story-volume
          persistentVolumeClaim:
            claimName: host-pvc
```

### **Key Enhancements**
- We increased **replicas to 2**, ensuring multiple instances of our application run.
- The **PVC (host-pvc)** is mounted inside the container at `/app/story`, making persistent storage accessible.

## **Why Persistent Volumes Matter in Multi-Node Clusters**

Unlike `hostPath`, which binds storage **to a single node**, Persistent Volumes ensure **data availability across multiple pods and nodes**. Even if a pod fails and Kubernetes reschedules it on a new node, the data remains intact.

### **Summary**

Moving from standard **Docker Volumes** to **Kubernetes Volumes** and finally to **Persistent Volumes** is essential for building scalable, cloud-ready applications. 

For further exploration, refer to the [official Kubernetes storage documentation](https://kubernetes.io/docs/concepts/storage/volumes/). 

---
With Persistent Volumes, your application's data **survives pod failures, rescheduling, and multi-node deployments**, making it **reliable for production environments**. Now you’re ready to manage **stateful applications at scale**.