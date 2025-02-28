# Kubernetes Resource Hierarchy

Below is a structured representation of Kubernetes resources and their hierarchy. This table and tree diagram will help you understand how resources are organized in a Kubernetes cluster.

---

## **1. Tree Diagram of Kubernetes Resources**

```plaintext
Cluster
├── Namespace (optional)
│   ├── Pod
│   │   ├── Container
│   │   ├── Volume
│   │   ├── Network Policy
│   ├── Service
│   ├── Deployment
│   ├── StatefulSet
│   ├── DaemonSet
│   ├── Job
│   ├── CronJob
│   ├── ConfigMap
│   ├── Secret
│   ├── Ingress
│   ├── PersistentVolumeClaim (PVC)
│
└── Cluster-Wide Resources
    ├── PersistentVolume (PV)
    ├── ClusterRole
    ├── ClusterRoleBinding
    ├── CustomResourceDefinition (CRD)
```

---

## **2. Table of Kubernetes Resources**

| **Resource Type**          | **Description**                                                                 | **Namespace-Scoped?** | **Cluster-Wide?** |
|----------------------------|---------------------------------------------------------------------------------|-----------------------|-------------------|
| **Namespace**              | Logical separation of resources within a cluster.                               | No                    | Yes               |
| **Pod**                    | Smallest deployable unit, containing one or more containers.                   | Yes                   | No                |
| **Container**              | A running process inside a Pod.                                                | Yes                   | No                |
| **Volume**                 | Provides storage to Pods.                                                      | Yes                   | No                |
| **Network Policy**         | Defines network rules for Pod communication.                                   | Yes                   | No                |
| **Service**                | Exposes Pods as a network service.                                             | Yes                   | No                |
| **Deployment**             | Manages stateless applications and ensures desired replicas are running.       | Yes                   | No                |
| **StatefulSet**            | Manages stateful applications with stable network identities and storage.      | Yes                   | No                |
| **DaemonSet**              | Ensures a copy of a Pod runs on all or specific nodes.                         | Yes                   | No                |
| **Job**                    | Runs a task to completion.                                                     | Yes                   | No                |
| **CronJob**                | Runs Jobs on a time-based schedule.                                            | Yes                   | No                |
| **ConfigMap**              | Stores non-sensitive configuration data.                                       | Yes                   | No                |
| **Secret**                 | Stores sensitive data like passwords or tokens.                                | Yes                   | No                |
| **Ingress**                | Manages external HTTP/HTTPS access to services.                                | Yes                   | No                |
| **PersistentVolumeClaim (PVC)** | Requests storage from a PersistentVolume.                                | Yes                   | No                |
| **PersistentVolume (PV)**  | Represents a piece of storage in the cluster.                                  | No                    | Yes               |
| **ClusterRole**            | Defines permissions for cluster-wide resources.                                | No                    | Yes               |
| **ClusterRoleBinding**     | Binds a ClusterRole to users or groups.                                        | No                    | Yes               |
| **CustomResourceDefinition (CRD)** | Defines custom resources in the cluster.                              | No                    | Yes               |

---

## **3. When to Use Namespace-Scoped vs Cluster-Wide Resources**

| **Resource Type**          | **When to Use**                                                                 | **When Not to Use**                       |
|----------------------------|---------------------------------------------------------------------------------|-------------------------------------------|
| **Namespace-Scoped**       | Resources that are specific to an application, team, or environment.            | For resources that need to be shared across the entire cluster. |
| **Cluster-Wide**           | Resources that are shared across the entire cluster (e.g., PVs, ClusterRoles).  | For resources that are specific to a namespace. |

---

## **4. Example YAML for Key Resources**

### **Namespace**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

### **Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace
spec:
  containers:
  - name: my-container
    image: nginx
```

### **Deployment**
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

### **Service**
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

### **PersistentVolume (PV)**
```yaml
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
```

### **PersistentVolumeClaim (PVC)**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: my-namespace
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
