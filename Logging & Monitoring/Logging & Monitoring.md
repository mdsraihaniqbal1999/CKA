# Kubernetes Logging and Monitoring (CKA Notes)

## 1. What is Logging and Monitoring?

### Logging
Logging is the process of collecting and storing system events, application messages, and system diagnostics. Logs help in debugging, performance tuning, and auditing application behavior.

### Monitoring
Monitoring involves observing the health, performance, and resource utilization of a system. It helps in detecting anomalies, forecasting capacity needs, and ensuring high availability.

---

## 2. Does Kubernetes Support Logging and Monitoring?

Yes, Kubernetes provides built-in logging and monitoring capabilities. However, it does not store logs persistently, so external tools are required for long-term storage and analysis.

### Logging in Kubernetes
- Logs can be accessed via `kubectl logs <pod-name>`.
- Logs are available only as long as the pod exists.
- Tools like Fluentd, Elasticsearch, and Loki can be used for centralized logging.

### Monitoring in Kubernetes
- Metrics can be obtained using `kubectl top node` and `kubectl top pod`.
- The Metrics Server provides resource usage statistics.
- Tools like Prometheus and Grafana enable advanced monitoring.

---

## 3. Kubernetes Monitoring Components

### Kubelet
- The Kubelet is an agent running on each node that ensures containers are running as expected.
- It exposes metrics about the node and its pods via the `/metrics` endpoint.

### cAdvisor (Container Advisor)
- cAdvisor is built into the Kubelet and collects resource usage data about running containers.
- It provides per-container CPU, memory, disk, and network usage statistics.

### Metrics Server
- The Metrics Server is a lightweight aggregator of resource metrics.
- It collects resource usage data from Kubelets and makes it accessible via the Kubernetes API.
- It enables commands like `kubectl top nodes` and `kubectl top pods`.

### Prometheus
- Prometheus scrapes metrics from various Kubernetes components.
- It stores time-series data and provides alerting capabilities.

### Grafana
- Grafana visualizes data collected by Prometheus and other sources.
- It provides dashboards for monitoring cluster performance.

---

## 4. What Components Are Monitored and How?

| Component       | Monitored Metrics | How It Works |
|---------------|----------------|----------------|
| Node | CPU, Memory, Disk Usage, Network | `kubectl top nodes` retrieves real-time data via the Metrics Server. |
| Pod | CPU, Memory Usage | `kubectl top pods` queries Metrics Server for pod-level resource usage. |
| Container | CPU, Memory, Network, Disk | cAdvisor collects per-container metrics and exposes them via the Kubelet. |
| Cluster | Overall Health, Resource Allocation | Prometheus scrapes data from all components and provides insights. |

---

## 5. When to Use What?

| Feature        | When to Use | When Not to Use |
|---------------|------------|----------------|
| `kubectl logs` | Debugging a single pod | When logs need to be persisted |
| Fluentd + Elasticsearch | Centralized logging for multiple clusters | Small applications with minimal logging needs |
| Prometheus    | Real-time monitoring & alerting | For long-term event logging |
| Metrics Server | Collecting resource usage (CPU, Memory) | If detailed historical trends are required |
| Grafana | Visualizing system health | If Prometheus is not set up |

---

## 6. Practical Example: Logging and Monitoring in a Kind Cluster

### Step 1: Create a Kind Cluster
```bash
kind create cluster --name logging-cluster --config kind-config.yaml
```
Example `kind-config.yaml`:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
```

### Step 2: Deploy an Event Simulator Pod for Logging
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
```
Apply the configuration:
```bash
kubectl apply -f event-simulator.yaml
```
Check logs:
```bash
kubectl logs -f event-simulator-pod
```

### Step 3: Enable the Kubernetes Metrics Server
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify the metrics server is working:
```bash
kubectl top nodes
kubectl top pods
```

### Step 4: Install Prometheus for Advanced Monitoring
```bash
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```
Access the Prometheus dashboard:
```bash
kubectl port-forward svc/prometheus-kube-prometheus-prometheus -n monitoring 9090:9090
```
Open `http://localhost:9090` in your browser.

---

## References
- [Kubernetes Logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [Kubernetes Monitoring](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
- [Prometheus and Grafana Setup](https://prometheus.io/docs/prometheus/latest/getting_started/)
- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
- [cAdvisor](https://github.com/google/cadvisor)


