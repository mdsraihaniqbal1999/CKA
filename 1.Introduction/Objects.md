# Kubernetes Objects: Notes for CKA Exam

Kubernetes objects are persistent entities in the Kubernetes system that represent the state of your cluster. They are used to define workloads, configurations, and policies. Below is a detailed breakdown of Kubernetes objects, their hierarchy, when to use them, and examples in YAML format.

---

## **1. Kubernetes Object Overview**

Kubernetes objects are defined using YAML or JSON manifests. Each object has:
- **apiVersion**: The version of the Kubernetes API.
- **kind**: The type of object (e.g., Pod, Deployment, Service).
- **metadata**: Data that identifies the object (e.g., name, labels).
- **spec**: Describes the desired state of the object.

---

## **2. Common Kubernetes Objects**

### **a) Pod**
- **What it is**: The smallest deployable unit in Kubernetes. A Pod can contain one or more containers.
- **When to use**:
  - For running a single instance of an application.
  - For testing or debugging purposes.
- **When not to use**:
  - For production workloads (use Deployments instead).
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
    - name: my-container
      image: nginx
  ```

---

### **b) Deployment**
- **What it is**: Manages a set of identical Pods and ensures the desired number of replicas are running.
- **When to use**:
  - For stateless applications.
  - When you need rolling updates or rollbacks.
- **When not to use**:
  - For stateful applications (use StatefulSets instead).
- **Example YAML**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: my-app
    template:
      metadata:
        labels:
          app: my-app
      spec:
        containers:
        - name: my-container
          image: nginx
  ```

---

### **c) Service**
- **What it is**: Exposes a set of Pods as a network service.
- **Types**:
  - **ClusterIP**: Exposes the service internally within the cluster.
  - **NodePort**: Exposes the service on a static port on each node.
  - **LoadBalancer**: Exposes the service externally using a cloud provider’s load balancer.
- **When to use**:
  - To provide stable network access to Pods.
  - To load balance traffic across Pods.
- **When not to use**:
  - If you don’t need network access to Pods.
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    selector:
      app: my-app
    ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
    type: ClusterIP
  ```

---

### **d) ConfigMap**
- **What it is**: Stores configuration data as key-value pairs.
- **When to use**:
  - To decouple configuration from application code.
  - To store non-sensitive configuration data.
- **When not to use**:
  - For storing sensitive data (use Secrets instead).
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-config
  data:
    key1: value1
    key2: value2
  ```

---

### **e) Secret**
- **What it is**: Stores sensitive data (e.g., passwords, tokens) in an encrypted format.
- **When to use**:
  - To store sensitive information like credentials or API keys.
- **When not to use**:
  - For non-sensitive data (use ConfigMaps instead).
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-secret
  type: Opaque
  data:
    username: dXNlcm5hbWU=  # base64 encoded
    password: cGFzc3dvcmQ=  # base64 encoded
  ```

---

### **f) Namespace**
- **What it is**: Provides a logical separation of resources within a cluster.
- **When to use**:
  - To isolate resources for different teams or environments.
  - To manage resource quotas and limits.
- **When not to use**:
  - For small clusters with limited resources.
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: my-namespace
  ```

---

### **g) PersistentVolume (PV) and PersistentVolumeClaim (PVC)**
- **What they are**:
  - **PV**: Represents a piece of storage in the cluster.
  - **PVC**: A request for storage by a user.
- **When to use**:
  - For stateful applications that require persistent storage.
- **When not to use**:
  - For stateless applications.
- **Example YAML**:
  ```yaml
  # PersistentVolume
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: my-pv
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: /mnt/data

  # PersistentVolumeClaim
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi
  ```

---

## **3. Hierarchy of Kubernetes Resources**

| **Resource**       | **Description**                                                                 | **Parent Resource**       |
|---------------------|---------------------------------------------------------------------------------|---------------------------|
| Pod                | Smallest deployable unit.                                                      | None                      |
| Deployment         | Manages Pods and ensures desired replicas are running.                         | Pod                       |
| Service            | Exposes Pods as a network service.                                             | Pod                       |
| ConfigMap          | Stores non-sensitive configuration data.                                       | Pod                       |
| Secret             | Stores sensitive data.                                                         | Pod                       |
| Namespace          | Provides logical separation of resources.                                      | None                      |
| PersistentVolume   | Represents storage in the cluster.                                             | PersistentVolumeClaim     |
| PersistentVolumeClaim | Requests storage from a PersistentVolume.                                   | Pod                       |

---

## **4. When to Use and Not Use Kubernetes Objects**

| **Object**         | **When to Use**                                                                 | **When Not to Use**                       |
|---------------------|---------------------------------------------------------------------------------|-------------------------------------------|
| Pod                | Testing, debugging, or single-instance applications.                          | Production workloads.                     |
| Deployment         | Stateless applications, rolling updates, rollbacks.                           | Stateful applications.                    |
| Service            | Exposing Pods to the network, load balancing.                                  | No network access needed.                 |
| ConfigMap          | Storing non-sensitive configuration data.                                      | Storing sensitive data.                   |
| Secret             | Storing sensitive data like passwords or tokens.                              | Storing non-sensitive data.               |
| Namespace          | Isolating resources for teams or environments.                                | Small clusters with limited resources.    |
| PersistentVolume   | Providing persistent storage for stateful applications.                        | Stateless applications.                   |

---

## **5. Example YAML Hierarchy**

Here’s an example of how resources are connected:

1. **Namespace**:
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: my-namespace
   ```

2. **Deployment** (inside the namespace):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-deployment
     namespace: my-namespace
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-container
           image: nginx
   ```

3. **Service** (exposes the Deployment):
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
     namespace: my-namespace
   spec:
     selector:
       app: my-app
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
     type: ClusterIP
   ```


