# 📌 Helm Chart for service.yaml and deployment.yaml

## 🚀 Introduction
This guide walks through creating a **Kind cluster** with **1 control plane and 2 worker nodes** and setting up a **Helm chart** for `service.yaml` and `deployment.yaml` using VS Code. We also cover Helm templating best practices.

---

## 🔧 1. Setting Up the Kind Cluster

### 📌 Step 1: Install Kind
Ensure you have [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) installed.
```sh
kind version
```

### 📌 Step 2: Create a Kind Cluster
Create a `kind-config.yaml` file:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```
Apply the configuration:
```sh
kind create cluster --config kind-config.yaml
```
Verify the cluster:
```sh
kubectl get nodes
```

---

## 📂 2. Creating a Helm Chart

### 📌 Step 1: Create the Chart
```sh
helm create mychart
cd mychart
```

### 📌 Step 2: Define Deployment (`templates/deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: myapp
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          ports:
            - containerPort: 80
```

### 📌 Step 3: Define Service (`templates/service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 80
```

### 📌 Step 4: Update `values.yaml`
```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
service:
  type: ClusterIP
  port: 80
```

---

## 📜 3. Understanding Helm Templating

| Concept | Description |
|---------|------------|
| `{{ .Release.Name }}` | Inserts the release name dynamically |
| `{{ .Values.key }}` | References values from `values.yaml` |
| `{{- if .Values.enabled }}` | Conditional rendering in templates |

Example of templating in `_helpers.tpl`:
```yaml
{{- define "app.name" -}}
{{ .Release.Name }}-app
{{- end -}}
```
Usage in `deployment.yaml`:
```yaml
metadata:
  name: {{ include "app.name" . }}
```

---

## 🚀 4. Deploying the Helm Chart

### 📌 Step 1: Package and Install
```sh
helm package mychart
helm install myapp ./mychart
```

### 📌 Step 2: Verify Deployment
```sh
kubectl get all
```

### 📌 Step 3: Uninstall
```sh
helm uninstall myapp
```

---

## ✅ Best Practices
| Best Practice | Why? |
|--------------|------|
| Use `_helpers.tpl` for reusability | Avoid duplication in templates |
| Reference `values.yaml` for configurations | Keep templates clean and dynamic |
| Version lock dependencies | Ensure compatibility with external charts |


