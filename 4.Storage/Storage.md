# Kubernetes Storage Management for CKA Exam

This guide covers storage concepts and practical implementations for the Certified Kubernetes Administrator (CKA) exam, focusing on persistent storage management in Kubernetes clusters.

## Table of Contents
- [Storage Classes](#storage-classes)
- [Dynamic Volume Provisioning](#dynamic-volume-provisioning)
- [Volume Types and Access Modes](#volume-types-and-access-modes)
- [Persistent Volumes and Persistent Volume Claims](#persistent-volumes-and-persistent-volume-claims)
- [Reclaim Policies](#reclaim-policies)
- [Connecting Storage to Pods](#connecting-storage-to-pods)
- [Practical Exercises](#practical-exercises)

## Storage Classes

Storage Classes provide a way to describe different "classes" of storage offered in a Kubernetes cluster. They act as a level of abstraction between storage infrastructure and how Kubernetes pods use it.

### Key Concepts

- **Provisioner**: Determines which volume plugin to use for provisioning PVs (e.g., AWS EBS, GCE PD, Azure Disk)
- **Parameters**: Configuration options specific to the provisioner
- **Reclaim Policy**: Controls what happens to PVs when their claims are deleted
- **Volume Binding Mode**: Determines when volume binding and dynamic provisioning occur

### Example StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### Common Storage Class Parameters

| Provisioner | Common Parameters | Example Values |
|-------------|-------------------|----------------|
| kubernetes.io/aws-ebs | type, zone | gp2, us-east-1a |
| kubernetes.io/gce-pd | type, zone | pd-standard, us-central1-a |
| kubernetes.io/azure-disk | storageaccounttype, kind | Standard_LRS, Managed |
| kubernetes.io/vsphere-volume | diskformat | thin, zeroedthick |
| kubernetes.io/no-provisioner | (For pre-provisioned volumes) | N/A |

## Dynamic Volume Provisioning

Dynamic volume provisioning allows storage volumes to be created on-demand when a PVC is created, eliminating the need to pre-provision storage.

### How It Works

1. Admin creates StorageClass(es) defining available storage types
2. User creates PVC that references a StorageClass
3. Dynamic provisioner watches for new PVCs and automatically creates PVs
4. PV is bound to the PVC and available for pods to use

### Example PVC for Dynamic Provisioning

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 5Gi
```

## Volume Types and Access Modes

### Common Volume Types

| Volume Type | Description | Use Case |
|-------------|-------------|----------|
| awsElasticBlockStore | AWS EBS volume | Persistent storage on AWS |
| azureDisk | Azure Disk | Persistent storage on Azure |
| gcePersistentDisk | GCE PD | Persistent storage on GCP |
| hostPath | Uses path on node's filesystem | Development/testing only (not for production) |
| nfs | NFS share mounted into pod | Shared storage across pods |
| persistentVolumeClaim | Points to a PV | Most common for production |
| emptyDir | Empty directory in pod | Temporary storage |
| configMap | Inject config data | Configuration files |
| secret | Inject secret data | Sensitive information |

### Access Modes

| Access Mode | Description | When to Use | When Not to Use |
|-------------|-------------|-------------|-----------------|
| ReadWriteOnce (RWO) | Volume can be mounted as read-write by a single node | Single pod access or pods on same node | Multiple pods on different nodes |
| ReadOnlyMany (ROX) | Volume can be mounted read-only by many nodes | Read-only data shared across pods | When data needs to be written |
| ReadWriteMany (RWX) | Volume can be mounted as read-write by many nodes | Shared data requiring write access from multiple pods | Not all volume types support this |
| ReadWriteOncePod (RWOP) | Volume can be mounted as read-write by a single pod (K8s 1.22+) | Exclusive access for a specific pod | Older Kubernetes versions |

### Access Mode Support by Volume Types

| Volume Plugin | ReadWriteOnce | ReadOnlyMany | ReadWriteMany | ReadWriteOncePod |
|---------------|---------------|--------------|---------------|------------------|
| AWS EBS | ✓ | ✗ | ✗ | ✓ |
| Azure Disk | ✓ | ✗ | ✗ | ✓ |
| GCE PD | ✓ | ✓ | ✗ | ✓ |
| NFS | ✓ | ✓ | ✓ | ✓ |
| iSCSI | ✓ | ✓ | ✗ | ✓ |
| Ceph RBD | ✓ | ✓ | ✗ | ✓ |
| CephFS | ✓ | ✓ | ✓ | ✓ |
| HostPath | ✓ | ✗ | ✗ | ✓ |

## Persistent Volumes and Persistent Volume Claims

### Persistent Volume (PV)

A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

#### Key Fields in PV Spec

| Field | Description | Example |
|-------|-------------|---------|
| capacity | Size of the volume | 10Gi |
| accessModes | How the volume can be mounted | [ReadWriteOnce] |
| persistentVolumeReclaimPolicy | What happens when PVC is deleted | Retain, Delete, Recycle |
| storageClassName | Name of StorageClass this PV belongs to | standard |
| mountOptions | Options for mounting the volume | [hard, nfsvers=4.1] |
| volumeMode | Block or Filesystem | Filesystem |
| nodeAffinity | Constraints for what nodes can access the volume | (Node selector terms) |

### Example PV Definition

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  nfs:
    path: /data
    server: nfs-server.example.com
```

### Persistent Volume Claim (PVC)

A request for storage by a user that can be fulfilled by a PV.

#### Key Fields in PVC Spec

| Field | Description | Example |
|-------|-------------|---------|
| accessModes | How the volume should be mounted | [ReadWriteOnce] |
| resources | How much storage is requested | requests: storage: 5Gi |
| storageClassName | What StorageClass to use | standard |
| selector | Label selector to filter PVs | matchLabels: environment: prod |
| volumeMode | Block or Filesystem | Filesystem |
| volumeName | Specific PV to bind to | pv-example |

### Example PVC Definition

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

## Reclaim Policies

Reclaim policies determine what happens to a Persistent Volume when its claim is deleted.

| Policy | Description | When to Use |
|--------|-------------|-------------|
| Retain | PV remains with its data, becomes "Released" but not available for another claim | When data must be preserved for manual recovery |
| Delete | PV and associated storage asset are deleted | When data can be discarded after use |
| Recycle (deprecated) | Basic scrub (rm -rf) is performed, then available for new claim | Legacy systems only, use dynamic provisioning instead |

## Connecting Storage to Pods

### Pod Volume Definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: app-container
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: my-volume
  volumes:
  - name: my-volume
    persistentVolumeClaim:
      claimName: pvc-example
```

### Key Components

1. **volumes**: Defines what volumes the pod has access to
2. **volumeMounts**: Specifies where in the container filesystem the volume should be mounted
3. **persistentVolumeClaim**: References an existing PVC by name

### Storage Workflow

```
StorageClass → PersistentVolume → PersistentVolumeClaim → Pod (volumes + volumeMounts)
```

## Practical Exercises



# Practical Kubernetes Storage Exercises

## Exercise 1: Create a Storage Class and Dynamically Provision a Volume

### 1. Create a StorageClass

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/no-provisioner  # Use appropriate provisioner for your environment
parameters:
  type: ssd
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
EOF
```

### 2. Create a PVC using the StorageClass

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 1Gi
EOF
```

### 3. Verify the PVC status

```bash
kubectl get pvc dynamic-pvc
```

### 4. Create a Pod using the PVC

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pvc-pod
spec:
  containers:
  - name: task-pv-container
    image: nginx
    ports:
    - containerPort: 80
      name: "http-server"
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: task-pv-storage
  volumes:
  - name: task-pv-storage
    persistentVolumeClaim:
      claimName: dynamic-pvc
EOF
```

### 5. Verify the Pod is using the PVC

```bash
kubectl describe pod dynamic-pvc-pod
```

## Exercise 2: Create a Static Persistent Volume and Bind it to a PVC

### 1. Create a Persistent Volume

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv
  labels:
    type: local
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
EOF
```

### 2. Create a PVC to bind to the PV

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
EOF
```

### 3. Verify the binding

```bash
kubectl get pv
kubectl get pvc
```

### 4. Create a Pod using the PVC

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: static-pvc-pod
spec:
  containers:
  - name: static-pv-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: static-pv-storage
  volumes:
  - name: static-pv-storage
    persistentVolumeClaim:
      claimName: static-pvc
EOF
```

### 5. Verify pod is running and using the volume

```bash
kubectl describe pod static-pvc-pod
```

## Exercise 3: Using Different Access Modes

### 1. Create a PV with ReadOnlyMany access mode

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: readonly-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data-readonly"
EOF
```

### 2. Create a PVC for the ReadOnlyMany PV

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: readonly-pvc
spec:
  accessModes:
    - ReadOnlyMany
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
EOF
```

### 3. Create multiple pods using the same PVC

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod-1
spec:
  containers:
  - name: readonly-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: readonly-storage
      readOnly: true
  volumes:
  - name: readonly-storage
    persistentVolumeClaim:
      claimName: readonly-pvc
---
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod-2
spec:
  containers:
  - name: readonly-container
    image: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - mountPath: "/data"
      name: readonly-storage
      readOnly: true
  volumes:
  - name: readonly-storage
    persistentVolumeClaim:
      claimName: readonly-pvc
EOF
```

### 4. Verify both pods are using the same volume

```bash
kubectl describe pod readonly-pod-1
kubectl describe pod readonly-pod-2
```

## Exercise 4: Testing Volume Expansion

### 1. Create a StorageClass that allows volume expansion

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable-sc
provisioner: kubernetes.io/no-provisioner  # Use appropriate provisioner for your environment
allowVolumeExpansion: true
EOF
```

### 2. Create a PVC using the expandable StorageClass

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: expandable-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: expandable-sc
  resources:
    requests:
      storage: 1Gi
EOF
```

### 3. Create a Pod using the PVC

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: expandable-pod
spec:
  containers:
  - name: volume-container
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: expandable-volume
  volumes:
  - name: expandable-volume
    persistentVolumeClaim:
      claimName: expandable-pvc
EOF
```

### 4. Expand the PVC

```bash
kubectl edit pvc expandable-pvc
```

Change the storage request from 1Gi to 2Gi:

```yaml
spec:
  resources:
    requests:
      storage: 2Gi
```

### 5. Check the status of the expansion

```bash
kubectl get pvc expandable-pvc
```

## Exercise 5: Testing Different Reclaim Policies

### 1. Create a PV with Retain policy

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: retain-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data-retain"
EOF
```

### 2. Create a PV with Delete policy

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: delete-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: manual
  hostPath:
    path: "/mnt/data-delete"
EOF
```

### 3. Create PVCs for each PV

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: retain-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
  volumeName: retain-pv
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: delete-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
  volumeName: delete-pv
EOF
```

### 4. Create pods using the PVCs

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: retain-pod
spec:
  containers:
  - name: volume-container
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: retain-volume
  volumes:
  - name: retain-volume
    persistentVolumeClaim:
      claimName: retain-pvc
---
apiVersion: v1
kind: Pod
metadata:
  name: delete-pod
spec:
  containers:
  - name: volume-container
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: delete-volume
  volumes:
  - name: delete-volume
    persistentVolumeClaim:
      claimName: delete-pvc
EOF
```

### 5. Test the reclaim policies

```bash
# Delete the pods
kubectl delete pod retain-pod delete-pod

# Delete the PVCs
kubectl delete pvc retain-pvc delete-pvc

# Check PV status
kubectl get pv
```

The retain-pv should be in "Released" state, while delete-pv should be deleted.


## Common Storage Troubleshooting

### Volume Mounting Issues

1. **PVC stuck in Pending state**
   - Check if a matching PV exists
   - Verify StorageClass exists and has correct provisioner
   - Check if cloud provider has necessary permissions

2. **Pod stuck in ContainerCreating with volume errors**
   - Check if PVC is bound
   - Verify node has access to the volume (especially for cloud volumes)
   - Check for zone/region mismatches

3. **Permission issues when writing to volumes**
   - Check filesystem permissions
   - Consider SecurityContext settings in pod spec
   - Verify SELinux or AppArmor settings if applicable

### Command Cheat Sheet

```bash
# List StorageClasses
kubectl get storageclass

# List PVs
kubectl get pv

# List PVCs
kubectl get pvc

# Describe a PV for details
kubectl describe pv <pv-name>

# Describe a PVC for details
kubectl describe pvc <pvc-name>

# Check pod volume mounts
kubectl describe pod <pod-name>

# Edit a StorageClass
kubectl edit storageclass <sc-name>

# Show PVs sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage

# List PVCs in all namespaces
kubectl get pvc --all-namespaces
```

## Best Practices

1. **Use StorageClasses and dynamic provisioning** whenever possible
2. **Choose appropriate access modes** based on workload needs
3. **Set appropriate reclaim policies** based on data importance
4. **Use volumeBindingMode: WaitForFirstConsumer** to avoid binding PVs until pods need them
5. **Implement storage quotas** to prevent excessive use
6. **Label PVs and PVCs** for better organization
7. **Consider volume expansion needs** when selecting StorageClasses
8. **Use appropriate volume type** for performance requirements
9. **Backup critical volumes** regularly using snapshot mechanisms
10. **Test failover scenarios** to ensure data availability

## Summary

- **Storage Classes** provide a way to define different types of storage with different capabilities
- **Dynamic provisioning** automates volume creation based on PVCs
- **Volume types** determine the actual storage backend (AWS EBS, GCE PD, NFS, etc.)
- **Access modes** control how volumes can be mounted (RWO, ROX, RWX, RWOP)
- **Persistent Volumes** are cluster resources representing storage
- **Persistent Volume Claims** are requests for storage that bind to PVs
- **Reclaim policies** determine what happens to volumes when claims are deleted
- **Pods connect to storage** via volumeMounts and volumes specifications

This guide covers the key concepts and practical implementations needed for the CKA exam's storage components. Be sure to practice the exercises in a test environment to solidify your understanding.
