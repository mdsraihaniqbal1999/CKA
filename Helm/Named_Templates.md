# Helm Named Templates: A Guide for CKA Exam 🚀

## 📌 Understanding Named Templates
Named templates in Helm allow reusability of code blocks within chart templates. They are useful for avoiding duplication and maintaining cleaner, modular Helm templates.

### 🔹 Why Use Named Templates?
- **Code Reusability**: Avoid repeating common structures.
- **Maintainability**: Makes Helm charts easier to read and update.
- **Modularization**: Allows breaking complex templates into smaller components.

---

## 🛠 Defining Named Templates
Named templates are usually defined in `_helpers.tpl` and can be referenced across other templates.

### 🔹 Example Named Template
```yaml
{{- define "mychart.labels" }}
app: {{ .Release.Name }}
env: {{ .Values.env | default "production" }}
{{- end }}
```
This template, named `mychart.labels`, defines common Kubernetes labels.

---

## 📌 Using Named Templates

### 1️⃣ `template` Function (Preferred Method)
The `template` function renders a named template and passes a context.

```yaml
metadata:
  labels:
    {{- template "mychart.labels" . }}
```
✅ This injects the defined labels into `metadata.labels`.

---

### 2️⃣ `include` Function
The `include` function is similar to `template`, but it **returns the result as a string**, allowing further processing.

```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```
✅ `nindent 4` ensures proper YAML formatting by adding four spaces.

#### 🔍 Key Differences: `template` vs `include`
| Feature     | `template` | `include` |
|------------|-----------|-----------|
| Returns    | Renders directly | Returns as a string |
| Processing | Cannot modify output | Can be modified using functions like `nindent` |
| Usage      | Simple insertion | Useful for formatted or inline usage |

---

## ⚠️ Indentation Issues & Fixes
Incorrect indentation can break YAML structure when using named templates. Use `nindent` or `indent` to fix formatting.

### 🔹 Example Issue (Incorrect Indentation)
```yaml
metadata:
  labels:
  {{- include "mychart.labels" . }}
```
❌ This results in misaligned YAML.

### ✅ Fix Using `nindent`
```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```
✅ `nindent 4` ensures correct indentation under `labels`.

---

## 🚀 Practical Example: Named Templates in Kind Cluster
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
├── templates/
│   ├── deployment.yaml
│   ├── _helpers.tpl
├── values.yaml
├── Chart.yaml
```

### 🔹 Define Named Template in `_helpers.tpl`
```yaml
{{- define "mychart.image" -}}
{{ .Values.image.repository }}:{{ .Values.image.tag | default "latest" }}
{{- end }}
```

### 🔹 Use Named Template in `deployment.yaml`
```yaml
spec:
  containers:
    - name: my-container
      image: {{ include "mychart.image" . }}
```

### 🔍 Example `values.yaml`
```yaml
image:
  repository: nginx
  tag: alpine
```
✅ The image will be set dynamically as `nginx:alpine`.

---

## 🤔 Tricky Questions on Helm Named Templates

### 1️⃣ What happens if a named template is not defined?
- **A)** Helm throws an error.
- **B)** Helm ignores it and continues.
- **C)** Helm replaces it with an empty string.

✅ **Answer:** A) Helm throws an error.

---

### 2️⃣ What is the difference between `template` and `include`?
✅ `include` returns a string, allowing further processing, while `template` directly renders output.

---

### 3️⃣ What does `nindent 4` do in this example?
```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```
- **A)** Adds a 4-space indentation to each line.
- **B)** Removes indentation.
- **C)** Causes an error.

✅ **Answer:** A) Adds a 4-space indentation.

---

### 4️⃣ Fix the following incorrect `include` usage:
```yaml
metadata:
  labels:
  {{- include "mychart.labels" . }}
```
✅ **Solution:**
```yaml
metadata:
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```

---

### 5️⃣ What happens if a named template is called inside itself?
- **A)** Helm throws a recursion error.
- **B)** Helm ignores the recursion.
- **C)** Helm allows it, but only once.

✅ **Answer:** A) Helm throws a recursion error.



