# Helm Conditionals: A Guide for CKA Exam 🚀

## 📌 Understanding Helm Conditionals
Helm uses Go templating for defining conditionals that control resource creation dynamically. Conditionals help make Helm charts flexible by allowing logic-based decisions.

### 🔹 Why Use Helm Conditionals?
- **Dynamic Configuration**: Customize deployments based on user-defined values.
- **Feature Toggles**: Enable or disable features dynamically.
- **Resource Optimization**: Avoid unnecessary resource creation.
- **Reusability**: Make Helm charts more adaptable across different environments.

### 🔹 Types of Helm Conditionals
#### 1️⃣ `if`, `else if`, `else`
Used to execute specific sections of YAML based on conditions.
```yaml
{{- if .Values.enableFeature }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
{{- end }}
```
✅ If `enableFeature: true` in `values.yaml`, the ConfigMap is created.
❌ If `false`, it is skipped.

#### 2️⃣ `with`
Used to simplify access to nested values.
```yaml
{{- with .Values.config }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .name }}
{{- end }}
```
✅ Allows direct access to `config.name` instead of `Values.config.name`.

#### 3️⃣ `has`
Checks if a key exists in a list or dictionary.
```yaml
{{- if has "nginx" .Values.enabledApps }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
{{- end }}
```
✅ Creates a deployment only if `enabledApps` contains `"nginx"`.

#### 4️⃣ Comparison Operators (`eq`, `ne`, `lt`, `gt`, etc.)
Used for value comparisons.
```yaml
{{- if eq .Values.replicaCount 3 }}
  replicas: 3
{{- else }}
  replicas: 1
{{- end }}
```
✅ Sets different replica counts based on `values.yaml`.

## 🛠 Practical Example: Helm Chart for Kind Cluster
### 🏗 Cluster Setup (1 Control Plane, 2 Workers)
Ensure you have a running Kind cluster:
```sh
kind create cluster --name helm-demo --config=kind-config.yaml
```
Example `kind-config.yaml`:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

### 📦 Helm Chart Structure
```sh
helm-demo/
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl
│   ├── NOTES.txt
├── values.yaml
├── Chart.yaml
```

### 🔹 Example: Conditional Service Type
Modify `service.yaml` to dynamically set `ClusterIP` or `LoadBalancer` based on values:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type | default "ClusterIP" }}
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: {{ .Release.Name }}
```

#### 🔍 Enabling LoadBalancer in `values.yaml`
```yaml
service:
  type: LoadBalancer  # Change to ClusterIP if needed
```

### 🔹 Example: Optional Environment Variables
Modify `deployment.yaml` to include optional ENV variables:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: my-app
          image: nginx
          env:
            {{- if .Values.envVars }}
            {{- range $key, $value := .Values.envVars }}
            - name: {{ $key }}
              value: "{{ $value }}"
            {{- end }}
            {{- end }}
```

#### 🔍 Enabling Environment Variables in `values.yaml`
```yaml
envVars:
  DEBUG_MODE: "true"
  LOG_LEVEL: "info"
```

## 🚀 Practical Exercise: Deploying the Helm Chart
1. **Install Helm Chart:**
   ```sh
   helm install myapp ./helm-demo --values values.yaml
   ```
2. **Check Resources:**
   ```sh
   kubectl get all
   ```
3. **Modify `values.yaml` and Upgrade:**
   ```sh
   helm upgrade myapp ./helm-demo --set service.type=ClusterIP
   ```
4. **Verify Changes:**
   ```sh
   kubectl get svc
   ```

## 🎯 Conclusion
- `if` conditions dynamically control values.
- `range` loops iterate over lists.
- `with` simplifies nested value access.
- This Helm chart can be modified for real-world scenarios.



