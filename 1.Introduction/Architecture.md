# Kubernetes Architecture Explained

## Introduction
Kubernetes is a powerful container orchestration platform that automates the deployment, scaling, and management of containerized applications. Understanding its architecture is essential for the CKA (Certified Kubernetes Administrator) exam.

## Core Components

<img width="747" alt="image" src="https://github.com/user-attachments/assets/3a81e4af-b3fa-49cb-ae4b-99088a0b6353" />


### 1. Control Plane Components
1. **kube-apiserver**
   - Acts as the front-end for the Kubernetes control plane
   - Exposes the Kubernetes API that users, CLI tools, and other components interact with
   - Validates and processes API requests
   - Serves as the gateway for communication between cluster components

2. **etcd**
   - Distributed key-value store that stores all cluster data
   - Acts as Kubernetes' persistent storage for configuration data
   - Maintains the state of the entire cluster
   - Critical for cluster stability (often deployed in highly available configurations)

3. **kube-scheduler**
   - Watches for newly created pods with no assigned node
   - Selects the best node for pod placement based on resource requirements
   - Considers factors like resource availability, constraints, affinity/anti-affinity rules
   - Doesn't actually place pods; it just decides where they should run

4. **kube-controller-manager**
   - Runs controller processes in a single binary
   - Key controllers include:
     - Node Controller: monitors node health and responds when nodes go down
     - Replication Controller: maintains the correct number of pod replicas
     - Endpoints Controller: populates the Endpoints object (joins Services & Pods)
     - Service Account & Token Controllers: create default accounts and API access tokens

5. **cloud-controller-manager** 
   - Interfaces with the underlying cloud provider
   - Allows cloud-specific code to be separated from Kubernetes core
   - Handles cloud-specific operations like creating load balancers, volumes, etc.
   - Only present when running Kubernetes on a cloud provider

### 2. Node Components
1. **kubelet**
   - Runs on every node in the cluster
   - Ensures containers are running in a Pod
   - Takes PodSpecs from the API server and ensures containers described are running/healthy
   - Reports node and pod status back to the control plane
   - Doesn't manage containers not created by Kubernetes

2. **kube-proxy**
   - Network proxy that runs on each node
   - Implements part of the Kubernetes Service concept
   - Maintains network rules (using iptables, IPVS) for pod communication
   - Handles internal network communication and load balancing across pods

3. **Container Runtime**
   - Software responsible for running containers
   - Kubernetes supports various runtimes via Container Runtime Interface (CRI)
   - Common options include containerd, CRI-O, and Docker (via dockershim, deprecated)
   - Handles container operations like pulling images and starting/stopping containers

## 3. Add-ons (Important for CKA)
1. **CoreDNS**
   - Provides DNS services within the cluster
   - Allows pods to resolve service names to cluster IPs
   - Enables service discovery within the cluster

2. **Ingress Controllers**
   - Manage external access to services within cluster
   - Provide HTTP/HTTPS routing, SSL termination, and name-based virtual hosting
   - Not part of base Kubernetes, must be deployed separately

3. **CNI Plugins (Container Network Interface)**
   - Responsible for network connectivity between pods
   - Implements Kubernetes networking model
   - Popular options include Calico, Flannel, Cilium, Weave Net

4. **Metrics Server**
   - Collects resource metrics from kubelet
   - Supports Horizontal Pod Autoscaler and Vertical Pod Autoscaler
   - Provides metrics for the `kubectl top` command

## 4. Kubernetes Objects & Resources
1. **Pods**
   - Smallest deployable units in Kubernetes
   - Group of one or more containers with shared storage/network
   - Ephemeral by nature (can be terminated and replaced)

2. **ReplicaSets**
   - Ensures specified number of pod replicas are running
   - Provides self-healing capabilities (recreates failed pods)
   - Usually managed by Deployments rather than directly

3. **Deployments**
   - Manages ReplicaSets and provides declarative updates
   - Supports rolling updates and rollbacks
   - Handles scaling and application lifecycle

4. **Services**
   - Provides stable networking for pods
   - Types include ClusterIP, NodePort, LoadBalancer, ExternalName
   - Enables service discovery and load balancing

5. **ConfigMaps & Secrets**
   - Store configuration data separate from application code
   - ConfigMaps for non-confidential data
   - Secrets for sensitive information (credentials, tokens)

6. **Volumes**
   - Provides persistent storage for pods
   - Many types: EmptyDir, HostPath, PVC, cloud provider volumes
   - Outlives individual containers, may outlive pods

7. **Namespaces**
   - Virtual clusters within a physical cluster
   - Provides scope for names and resource isolation
   - Helps organize resources in multi-tenant environments

## 5. Communication Paths
1. **Kubernetes API Communication**
   - All components communicate through the API server
   - Uses REST API over HTTPS
   - Authentication and authorization enforced at API level

2. **Node-to-Control Plane**
   - Nodes communicate with API server using HTTPS
   - Kubelets authenticate to the API server using certificates
   - API server pushes updates to nodes via watch mechanism

3. **Control Plane Internal Communication**
   - API server is the hub for all control plane communication
   - etcd only communicates with API server directly
   - Controllers and scheduler watch API server for changes

4. **Pod-to-Pod Communication**
   - Direct communication via cluster network
   - Every pod has unique IP address in flat network space
   - Pods on same node communicate via local bridge
   - Pods on different nodes communicate via overlay network

## 6. High Availability Setup
1. **Control Plane Redundancy**
   - Multiple control plane nodes (minimum 3 for HA)
   - Distributed etcd cluster (odd number of instances)
   - Load balancer in front of API servers

2. **etcd Considerations**
   - Requires quorum (N/2+1) for writes
   - Typically deployed as 3, 5, or 7 member clusters
   - Uses Raft consensus algorithm

3. **Node Redundancy**
   - Multiple worker nodes across availability zones
   - Pod anti-affinity to distribute workloads

## 7. Authentication and Authorization
1. **Authentication Methods**
   - Client certificates
   - Bearer tokens
   - OpenID Connect
   - Service accounts for internal components

2. **Authorization Mechanisms**
   - RBAC (Role-Based Access Control)
   - Node authorization
   - Attribute-based access control (ABAC)
   - Webhook mode

3. **Admission Control**
   - Intercepts requests after authentication but before persistence
   - Multiple admission controllers can modify or reject requests
   - Examples: ResourceQuota, LimitRanger, PodSecurityPolicy

This comprehensive breakdown of Kubernetes architecture covers the key components, objects, and communication patterns you'll need to understand for the CKA exam. Focus on understanding how these components interact to form a complete system.
