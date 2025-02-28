# Kubernetes (CKA) Command Cheatsheet

This comprehensive cheatsheet covers all essential commands you'll need for the Certified Kubernetes Administrator (CKA) exam, organized by category with explanations of when to use each command.

## Table of Contents
- [Core kubectl Commands](#core-kubectl-commands)
- [Pod Management](#pod-management)
- [Deployment Management](#deployment-management)
- [Service & Networking](#service--networking)
- [Configuration](#configuration)
- [Storage](#storage)
- [Authentication & Authorization](#authentication--authorization)
- [Node Management](#node-management)
- [Cluster Management](#cluster-management)
- [Troubleshooting Commands](#troubleshooting-commands)
- [Logging & Monitoring](#logging--monitoring)
- [Resource Consumption](#resource-consumption)
- [API Resources Reference](#api-resources-reference)
- [Exam-Specific Tips](#exam-specific-tips)
- [Command Generation Shortcuts](#command-generation-shortcuts)

## Core kubectl Commands

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl get <resource>` | List resources | When you need to see what resources exist |
| `kubectl describe <resource> <name>` | Show detailed information | When you need to inspect a resource's details |
| `kubectl create -f <file.yaml>` | Create resource from file | When creating resources from YAML manifests |
| `kubectl apply -f <file.yaml>` | Create/update resources | When applying changes to existing resources |
| `kubectl delete <resource> <name>` | Delete resources | When removing resources from the cluster |
| `kubectl edit <resource> <name>` | Edit resources in-place | When making quick changes to existing resources |
| `kubectl explain <resource>` | Documentation for resources | When you need help with resource fields |
| `kubectl config get-contexts` | List available contexts | When working with multiple clusters |
| `kubectl config use-context <name>` | Switch between contexts | When changing between clusters |
| `kubectl api-resources` | List all resources types | When you need to find API resource names |

## Pod Management

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl run <name> --image=<image>` | Create a pod | When quickly deploying a single pod |
| `kubectl get pods` | List all pods | When checking pod status |
| `kubectl get pods -o wide` | List pods with more details | When you need node and IP information |
| `kubectl get pods --show-labels` | Show pod labels | When working with label selectors |
| `kubectl get pods -l key=value` | Filter pods by label | When finding pods with specific labels |
| `kubectl describe pod <name>` | Inspect pod details | When troubleshooting pod issues |
| `kubectl logs <pod>` | View pod logs | When checking application output |
| `kubectl logs -f <pod>` | Stream pod logs | When monitoring real-time application logs |
| `kubectl logs <pod> -c <container>` | View specific container logs | When working with multi-container pods |
| `kubectl exec -it <pod> -- <command>` | Execute command in pod | When debugging or running commands in a pod |
| `kubectl port-forward <pod> <local>:<remote>` | Forward pod port | When accessing pod services directly |
| `kubectl delete pod <name>` | Delete a pod | When removing a specific pod |
| `kubectl delete pod <name> --grace-period=0 --force` | Force delete a pod | When a pod is stuck in terminating state |

## Deployment Management

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl create deployment <name> --image=<image>` | Create deployment | When deploying applications with replicas |
| `kubectl get deployments` | List deployments | When checking deployment status |
| `kubectl describe deployment <name>` | Show deployment details | When troubleshooting deployments |
| `kubectl scale deployment <name> --replicas=<n>` | Scale deployment | When changing number of pod replicas |
| `kubectl set image deployment/<name> <container>=<image>` | Update container image | When updating the application version |
| `kubectl rollout status deployment/<name>` | Check rollout status | When monitoring deployment updates |
| `kubectl rollout history deployment/<name>` | View rollout history | When checking previous deployment versions |
| `kubectl rollout undo deployment/<name>` | Rollback deployment | When reverting to previous version |
| `kubectl rollout restart deployment/<name>` | Restart deployment | When you need to refresh all pods |

## Service & Networking

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl expose <resource> <name> --port=<port>` | Create a service | When exposing applications internally |
| `kubectl get services` | List services | When checking service endpoints |
| `kubectl describe service <name>` | Show service details | When troubleshooting service connectivity |
| `kubectl get endpoints <name>` | List service endpoints | When verifying service-to-pod connections |
| `kubectl get networkpolicies` | List network policies | When checking network security rules |
| `kubectl get ingress` | List ingresses | When checking external access rules |
| `kubectl describe ingress <name>` | Show ingress details | When troubleshooting external access |
| `kubectl port-forward svc/<name> <local>:<svc>` | Forward service port | When accessing services for testing |

## Configuration

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl create configmap <name> --from-file=<path>` | Create ConfigMap from file | When creating configuration from files |
| `kubectl create configmap <name> --from-literal=key=value` | Create ConfigMap from literals | When creating simple configurations |
| `kubectl get configmaps` | List ConfigMaps | When checking available configurations |
| `kubectl describe configmap <name>` | Show ConfigMap details | When checking configuration values |
| `kubectl create secret generic <name> --from-literal=key=value` | Create Secret | When creating sensitive configurations |
| `kubectl get secrets` | List Secrets | When checking available secrets |
| `kubectl describe secret <name>` | Show Secret metadata | When checking secret metadata (not values) |
| `kubectl create quota <name>` | Create ResourceQuota | When setting resource limits for namespaces |

## Storage

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl get pv` | List PersistentVolumes | When checking available storage resources |
| `kubectl get pvc` | List PersistentVolumeClaims | When checking storage requests |
| `kubectl describe pv <name>` | Show PV details | When troubleshooting storage issues |
| `kubectl describe pvc <name>` | Show PVC details | When checking storage binding status |
| `kubectl get storageclass` | List StorageClasses | When checking available storage options |
| `kubectl patch pv <name> -p '{"spec":{"claimRef": null}}'` | Reclaim a PV | When unbinding a PV from a PVC |

## Authentication & Authorization

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl create serviceaccount <name>` | Create ServiceAccount | When setting up pod identities |
| `kubectl get serviceaccounts` | List ServiceAccounts | When checking available service accounts |
| `kubectl create role <name> --verb=<verb> --resource=<resource>` | Create Role | When setting up namespace-scoped permissions |
| `kubectl create clusterrole <name> --verb=<verb> --resource=<resource>` | Create ClusterRole | When setting up cluster-wide permissions |
| `kubectl create rolebinding <name> --role=<role> --user=<user>` | Create RoleBinding | When assigning namespace roles to users |
| `kubectl create clusterrolebinding <name> --clusterrole=<role> --user=<user>` | Create ClusterRoleBinding | When assigning cluster roles to users |
| `kubectl auth can-i <verb> <resource>` | Check permissions | When verifying your own permissions |
| `kubectl auth can-i <verb> <resource> --as=<user>` | Check user permissions | When verifying another user's permissions |
| `kubectl get roles` | List Roles | When checking namespace-scoped permissions |
| `kubectl get clusterroles` | List ClusterRoles | When checking cluster-wide permissions |
| `kubectl get rolebindings` | List RoleBindings | When checking role assignments |
| `kubectl get clusterrolebindings` | List ClusterRoleBindings | When checking cluster role assignments |

## Node Management

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl get nodes` | List nodes | When checking cluster nodes |
| `kubectl describe node <name>` | Show node details | When troubleshooting node issues |
| `kubectl drain <node>` | Drain a node | When performing maintenance (evicts pods) |
| `kubectl cordon <node>` | Cordon a node | When marking node as unschedulable |
| `kubectl uncordon <node>` | Uncordon a node | When returning node to service |
| `kubectl taint node <name> <key>=<value>:<effect>` | Add node taint | When controlling pod scheduling |
| `kubectl label node <name> <key>=<value>` | Label a node | When organizing nodes |
| `kubectl top node` | Show node resource usage | When checking node performance |

## Cluster Management

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl get namespaces` | List namespaces | When checking available namespaces |
| `kubectl create namespace <name>` | Create namespace | When creating resource isolation |
| `kubectl config set-context --current --namespace=<ns>` | Change namespace | When switching between namespaces |
| `kubectl get componentstatuses` | Check control plane status | When checking cluster health |
| `kubectl get events` | List cluster events | When troubleshooting recent issues |
| `kubectl get events --sort-by='.lastTimestamp'` | List events by time | When analyzing chronological issues |
| `kubectl version` | Show kubectl and server version | When checking compatibility |
| `kubectl cluster-info` | Show cluster info | When checking control plane endpoints |
| `kubectl api-versions` | List supported API versions | When checking API compatibility |

## Troubleshooting Commands

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl get pods -n <namespace> --field-selector=status.phase=Failed` | List failed pods | When finding problematic pods |
| `kubectl describe pod <pod> -n <namespace>` | Inspect pod status | When diagnosing pod issues |
| `kubectl logs <pod> -n <namespace> --previous` | View previous pod logs | When analyzing crashed containers |
| `kubectl get events --sort-by=.metadata.creationTimestamp` | View sorted events | When troubleshooting in chronological order |
| `kubectl get endpoints <service>` | Check service endpoints | When debugging service connectivity |
| `kubectl debug node/<name> -it --image=busybox` | Debug node | When troubleshooting node issues (K8s v1.18+) |
| `kubectl describe node <node> | grep Taint` | Check node taints | When debugging scheduling issues |
| `kubectl -n <namespace> get events --field-selector involvedObject.name=<pod>` | Get pod-specific events | When troubleshooting specific pod |

## Logging & Monitoring

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl top pods` | Show pod resource usage | When checking pod performance |
| `kubectl top pods -n <namespace>` | Show pod usage by namespace | When monitoring namespace resources |
| `kubectl logs -f <pod> -n <namespace>` | Stream pod logs | When monitoring real-time application behavior |
| `kubectl logs --tail=50 <pod>` | Show recent pod logs | When checking recent application activity |
| `kubectl logs <pod> -c <container>` | View specific container logs | When working with multi-container pods |

## Resource Consumption

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl top nodes` | Show node resource consumption | When identifying overloaded nodes |
| `kubectl top nodes --sort-by=cpu` | Sort nodes by CPU usage | When finding CPU-intensive nodes |
| `kubectl top nodes --sort-by=memory` | Sort nodes by memory usage | When finding memory-intensive nodes |
| `kubectl top pods --all-namespaces` | Show all pod resource usage | When monitoring cluster-wide consumption |
| `kubectl top pods --all-namespaces --sort-by=cpu` | Sort all pods by CPU | When finding CPU-intensive pods |
| `kubectl top pods --all-namespaces --sort-by=memory` | Sort all pods by memory | When finding memory-intensive pods |
| `kubectl top pods -n <namespace> --sort-by=cpu` | Sort namespace pods by CPU | When finding namespace resource hogs |
| `kubectl describe nodes | grep -A 5 "Allocated resources"` | Show node allocations | When checking node resource allocation |
| `kubectl get pods -o custom-columns=NAME:.metadata.name,CPU:.spec.containers[*].resources.requests.cpu,MEM:.spec.containers[*].resources.requests.memory` | Show pod resource requests | When checking resource allocations |
| `kubectl get pods -o custom-columns=NAME:.metadata.name,CPU:.spec.containers[*].resources.limits.cpu,MEM:.spec.containers[*].resources.limits.memory` | Show pod resource limits | When checking resource constraints |

## API Resources Reference

| Resource Type | Short Name | API Group | Namespaced | Description | When to Use |
|---------------|------------|-----------|------------|-------------|------------|
| `configmaps` | `cm` | v1 | Yes | Store configuration data | When decoupling config from pod specs |
| `persistentvolumeclaims` | `pvc` | v1 | Yes | Request storage resources | When requesting persistent storage |
| `persistentvolumes` | `pv` | v1 | No | Represent storage in cluster | When providing cluster-wide storage |
| `pods` | `po` | v1 | Yes | Smallest deployable units | When running containers |
| `replicationcontrollers` | `rc` | v1 | Yes | Ensure pod count | When maintaining identical pods (legacy) |
| `secrets` | | v1 | Yes | Store sensitive data | When managing sensitive information |
| `services` | `svc` | v1 | Yes | Expose pods as network service | When providing stable endpoints |
| `daemonsets` | `ds` | apps/v1 | Yes | Run pods on all/select nodes | When running background processes |
| `deployments` | `deploy` | apps/v1 | Yes | Declarative pod updates | When deploying applications |
| `replicasets` | `rs` | apps/v1 | Yes | Maintain stable replica count | When ensuring pod counts |
| `statefulsets` | `sts` | apps/v1 | Yes | Manage stateful applications | When maintaining pod identity |
| `cronjobs` | `cj` | batch/v1 | Yes | Run jobs on schedule | When running periodic tasks |
| `jobs` | | batch/v1 | Yes | Run one-off tasks | When running batch processes |
| `ingresses` | `ing` | networking.k8s.io/v1 | Yes | Manage external access | When exposing HTTP/HTTPS routes |
| `networkpolicies` | `netpol` | networking.k8s.io/v1 | Yes | Pod network isolation | When controlling pod traffic |
| `clusterroles` | | rbac.authorization.k8s.io/v1 | No | Cluster-wide permissions | When defining cluster-level permissions |
| `clusterrolebindings` | | rbac.authorization.k8s.io/v1 | No | Bind users to clusterroles | When assigning cluster-level permissions |
| `roles` | | rbac.authorization.k8s.io/v1 | Yes | Namespace permissions | When defining namespace-level permissions |
| `rolebindings` | | rbac.authorization.k8s.io/v1 | Yes | Bind users to roles | When assigning namespace permissions |
| `storageclasses` | `sc` | storage.k8s.io/v1 | No | Define storage classes | When defining storage provisioning types |

**Common API Resources Commands:**

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl api-resources` | List all API resources | When needing resource types and short names |
| `kubectl api-resources --namespaced=true` | List namespaced resources | When checking what can exist in a namespace |
| `kubectl api-resources --namespaced=false` | List cluster-wide resources | When working with cluster-level resources |
| `kubectl api-resources -o name` | List resource names only | When scripting against resources |
| `kubectl api-resources --verbs=list,get` | Filter resources by verb | When checking what you can view |
| `kubectl api-resources -o wide` | Show more details | When needing API group information |
| `kubectl api-resources --api-group=apps` | Filter by API group | When working with specific resource types |

## Exam-Specific Tips

| Command/Action | Description | When to Use |
|---------|-------------|------------|
| `kubectl config view` | View kubeconfig | When checking cluster access configuration |
| `export KUBECONFIG=<path>` | Set kubeconfig file | When switching between cluster configurations |
| `kubectl get all -n <namespace>` | Get all resources | When needing a quick overview of a namespace |
| `kubectl create -f <file.yaml> --dry-run=client -o yaml` | Test resource creation | When verifying YAML before applying |
| `kubectl run <name> --image=<image> --dry-run=client -o yaml > pod.yaml` | Generate pod YAML | When creating pod templates |
| `kubectl explain pods.spec` | Get documentation | When needing help with resource definitions |
| `alias k=kubectl` | Create kubectl alias | When speeding up command typing |
| `kubectx` | Switch contexts | When working with multiple clusters (if installed) |
| `kubens` | Switch namespaces | When working between namespaces (if installed) |

## Command Generation Shortcuts

| Command | Description | When to Use |
|---------|-------------|------------|
| `kubectl create deployment <name> --image=<image> --dry-run=client -o yaml` | Generate deployment YAML | When creating deployment templates |
| `kubectl expose pod <pod> --port=<port> --name=<name> --dry-run=client -o yaml` | Generate service YAML | When creating service templates |
| `kubectl create job <name> --image=<image> --dry-run=client -o yaml` | Generate job YAML | When creating job templates |
| `kubectl create configmap <name> --from-file=<path> --dry-run=client -o yaml` | Generate ConfigMap YAML | When creating ConfigMap templates |
| `kubectl create secret generic <name> --from-literal=key=value --dry-run=client -o yaml` | Generate Secret YAML | When creating Secret templates |

