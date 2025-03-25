# Kustomize Overlays: A Detailed Guide 🚀

## 📌 Overview
Kustomize overlays allow you to create **environment-specific configurations** while keeping a common base. They help avoid YAML duplication and make configurations modular and reusable.

## 🏗️ What Are Overlays?
Overlays extend a **base configuration** by modifying, adding, or removing resources for specific environments (e.g., dev, staging, production).

### 📂 Example Folder Structure
```
kustomize-overlays/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── replicas-patch.yaml
│   ├── prod/
│   │   ├── kustomization.yaml
│   │   ├── image-patch.yaml
```

## ✅ When to Use Overlays
- When managing **multiple environments** (dev, staging, prod) with **minor differences**.
- When applying **custom patches** to specific environments.
- When maintaining **a single source of truth** for Kubernetes manifests.
- When avoiding Helm and preferring a **pure YAML** approach.

## ❌ When NOT to Use Overlays
- If the differences between environments are **too significant**, requiring separate configurations.
- If Helm provides **a better templating solution** for the use case.
- If a simple **transformer** (like changing an image or namespace) is enough.

---

# 🔥 Process to Create and Apply Overlays
### 📌 Steps to Create Overlays
1. **Create a Base Directory**: Store the common Kubernetes resources.
2. **Define Overlays**: Create separate directories for each environment.
3. **Add Patches**: Modify specific fields per environment.
4. **Apply Overlays**: Use `kubectl apply -k` to deploy.
5. **Validate Changes**: Check if the expected modifications are applied.

## 📌 Step 1: Create a Kind Cluster 🏗️
```sh
cat <<EOF | kind create cluster --name=kustomize-overlays --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```

## 📌 Step 2: Define Base Configuration 📜
Create `kustomization.yaml` in `kustomize-overlays/base/`:
```yaml
resources:
  - deployment.yaml
  - service.yaml
```

Create `deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

## 📌 Step 3: Create Overlays 🎯

### ✅ Development Overlay (Modify Replicas) 🩹
Create `kustomization.yaml` in `kustomize-overlays/overlays/dev/`:
```yaml
bases:
  - ../../base
patchesStrategicMerge:
  - replicas-patch.yaml
```

Create `replicas-patch.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
```

### ✅ Production Overlay (Modify Image) 🖼️
Create `kustomization.yaml` in `kustomize-overlays/overlays/prod/`:
```yaml
bases:
  - ../../base
patchesStrategicMerge:
  - image-patch.yaml
```

Create `image-patch.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

## 📌 Step 4: Apply Overlays 🚀
```sh
kubectl apply -k kustomize-overlays/overlays/dev/
kubectl apply -k kustomize-overlays/overlays/prod/
```

## 📌 Step 5: Verify Deployment 🔍
```sh
kubectl get deployments -o yaml | grep replicas
kubectl get deployments -o yaml | grep image
```

## 📌 Step 6: Clean Up 🗑️
```sh
kind delete cluster --name=kustomize-overlays
```

---

# 🎯 Best Practices for Kustomize Overlays
| Best Practice | Description |
|--------------|-------------|
| **Use overlays for environment-specific configurations** 🌍 | Keep common resources in the base directory. |
| **Minimize duplication** 📌 | Only override the necessary fields in overlays. |
| **Use patches effectively** 🩹 | Apply **Strategic Merge** for simple changes and **JSON 6902** for complex modifications. |
| **Organize overlays properly** 📂 | Use separate directories for each environment. |
| **Validate before applying** ✅ | Run `kubectl kustomize` to preview changes. |

---

# 📚 CKA Exam Tips 🎓
✅ **Understand the relationship between base and overlays.** <br/>
✅ **Practice modifying resources using overlays in different environments.** <br/>
✅ **Use `kubectl apply -k` instead of `kubectl apply -f`.** <br/>
✅ **Optimize YAML structures to avoid excessive overlays.** <br/>
✅ **Be ready to modify deployments dynamically in the exam.** <br/>


