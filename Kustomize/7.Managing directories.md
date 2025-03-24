# 🚢 Kustomize Directory Management for Kubernetes 🌟

## 📂 Directory Management Overview

### When to Use Multiple Directories in Kustomize

| Scenario | Use Kustomize Directories | 🎯 Benefit |
|----------|----------------------------|------------|
| Multiple Microservices | ✅ Separate directories for each service | Modular configuration |
| Environment-specific Configs | ✅ Separate base and overlay directories | Easy environment management |
| Complex Applications | ✅ Hierarchical directory structure | Simplified configuration |

### 🚫 When NOT to Use Multiple Directories

| Scenario | Reason | 🚨 Caution |
|----------|--------|------------|
| Simple, Single Service | Unnecessary complexity | Overengineering |
| Small Projects | Adds overhead | Maintenance burden |
| Static Configurations | No environment variations | Redundant abstraction |

## 🏗️ Kustomize Directory Structure Best Practices

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── api-deployment.yaml
│   └── api-service.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patches/
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patches/
    └── prod/
        ├── kustomization.yaml
        └── patches/
```

## 📝 Key Kustomize Directory Management Techniques

### 1. 🔗 Base Directory
- Contains common, shared configurations
- Defines base resources applicable across environments
- Minimal, generic configuration

### 2. 🌈 Overlay Directories
- Environment-specific modifications
- Patch base configurations
- Add/modify resources for specific environments

## 🚀 Practical Example with Kind Cluster

### Cluster Creation
```bash
kind create cluster --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```

### Kustomize Directory Setup
```bash
mkdir -p k8s/base k8s/overlays/{dev,staging,prod}
```

### Base Kustomization (k8s/base/kustomization.yaml)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- nginx-deployment.yaml
- nginx-service.yaml
```

### Development Overlay (k8s/overlays/dev/kustomization.yaml)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
patches:
- patch: |-
    - op: replace
      path: /spec/replicas
      value: 2
```

## 🛠️ Deployment Steps
```bash
# Build and apply dev configuration
kustomize build k8s/overlays/dev | kubectl apply -f -
```

## 💡 Pro Tips for CKA Exam
- Practice creating nested Kustomize structures
- Understand patch strategies (strategic merge vs JSON 6902)
- Learn imperative Kustomize commands
- Master resource transformation techniques

## 🚧 Common Pitfalls
- Overcomplicate directory structures
- Ignore base configuration principles
- Misuse of patches and transformers

## 📊 Performance Considerations
- Minimal overhead with proper structuring
- Reduces configuration duplication
- Enhances maintainability

## 🔍 Exam Preparation
- Create mock microservice scenarios
- Practice environment-specific configurations
- Understand `kustomize build` and `kubectl apply -k`

### 🏆 Recommended Practice
1. Create base configurations
2. Add environment-specific overlays
3. Use patches for transformations
4. Test across different environments
