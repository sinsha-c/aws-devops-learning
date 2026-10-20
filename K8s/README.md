# Kubernetes for Beginners + Interview Q&A

A beginner-friendly guide to Kubernetes (K8s): what it is, why it is used, its core building blocks, hands-on commands, and 50 interview questions with answers.

---

## Table of Contents

1. [What is Kubernetes?](#1-what-is-kubernetes)
2. [Why Kubernetes is Used](#2-why-kubernetes-is-used)
3. [Kubernetes Architecture](#3-kubernetes-architecture)
4. [Core Concepts](#4-core-concepts)
5. [Configuration and Storage](#5-configuration-and-storage)
6. [Other Important Objects](#6-other-important-objects)
7. [How It Works End to End](#7-how-it-works-end-to-end)
8. [Hands-On: Your First Deployment](#8-hands-on-your-first-deployment)
9. [Essential kubectl Commands](#9-essential-kubectl-commands)
10. [Interview Questions and Answers](#10-interview-questions-and-answers)
11. [Quick Revision Table](#11-quick-revision-table)

---

## 1. What is Kubernetes?

**Kubernetes (K8s)** is an open-source **container orchestration platform**. It automates deploying, scaling, networking, and managing containerized applications across many machines.

It was originally built by Google and is now maintained by the **CNCF** (Cloud Native Computing Foundation).

> **Analogy:** Docker is a single worker. Kubernetes is the manager who hires workers, replaces absent ones, assigns tasks, and makes sure the work never stops.

---

## 2. Why Kubernetes is Used

Running 1-2 Docker containers is easy. Running hundreds across many servers is not. Kubernetes solves these problems:

| Problem | Kubernetes Solution |
|---|---|
| A container crashes | **Self-healing**: restarts or replaces it automatically |
| Traffic increases | **Auto-scaling**: adds more Pods (HPA) |
| New version release | **Rolling updates**: zero downtime |
| Bad release | **Rollback**: return to the previous version |
| Containers need to find each other | **Service discovery and DNS** |
| Uneven traffic | **Load balancing** |
| Vendor lock-in | **Portable**: works on AWS (EKS), Azure (AKS), GCP (GKE), on-premises |

**Key idea: declarative configuration.** You describe the *desired state* in YAML ("I want 3 copies of this app"), and Kubernetes continuously works to keep the *actual state* equal to it.

---

## 3. Kubernetes Architecture

A **Cluster** is a group of machines (nodes) that run your containerized applications. It has two parts:

### Control Plane (the brain)

| Component | Job |
|---|---|
| **kube-apiserver** | Front door of the cluster. Every `kubectl` command goes through it |
| **etcd** | Key-value database that stores the entire cluster state |
| **kube-scheduler** | Decides which node a new Pod should run on |
| **kube-controller-manager** | Watches the cluster and fixes any difference from the desired state |
| **cloud-controller-manager** | Connects Kubernetes to cloud provider features (load balancers, disks) |

### Worker Nodes (the muscle)

| Component | Job |
|---|---|
| **kubelet** | Agent on each node; makes sure Pods are running and healthy |
| **kube-proxy** | Maintains network rules so Pods and Services can communicate |
| **Container runtime** | Actually runs containers (containerd, CRI-O) |

```
        ┌──────────────────── CLUSTER ────────────────────┐
        │                                                 │
        │  CONTROL PLANE                                  │
        │  ┌───────────┐ ┌──────┐ ┌───────────┐ ┌──────┐  │
        │  │ API Server│ │ etcd │ │ Scheduler │ │ Ctrl │  │
        │  └───────────┘ └──────┘ └───────────┘ └──────┘  │
        │                                                 │
        │  WORKER NODES                                   │
        │  ┌──────────────┐     ┌──────────────┐          │
        │  │ Node 1       │     │ Node 2       │   ...    │
        │  │ kubelet      │     │ kubelet      │          │
        │  │ kube-proxy   │     │ kube-proxy   │          │
        │  │ [Pod] [Pod]  │     │ [Pod] [Pod]  │          │
        │  └──────────────┘     └──────────────┘          │
        └─────────────────────────────────────────────────┘
```

---

## 4. Core Concepts

### 4.1 Pod
- The **smallest deployable unit** in Kubernetes
- Wraps **one or more containers** that share the same network (IP) and storage
- Pods are **ephemeral**: if one dies, a replacement is created with a **new IP**

### 4.2 ReplicaSet
- Ensures a **specified number of identical Pods** are always running
- Usually managed by a Deployment, rarely created directly

### 4.3 Deployment
- The most common way to run stateless applications
- Provides **scaling, rolling updates, and rollback**
- Hierarchy: `Deployment → ReplicaSet → Pods → Containers`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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
        - name: web
          image: nginx:1.25
          ports:
            - containerPort: 80
```

### 4.4 Service
Pod IPs keep changing, so a **Service** provides a **stable IP and DNS name** and load-balances traffic across matching Pods (found using **labels**).

| Type | Purpose |
|---|---|
| **ClusterIP** (default) | Internal access only, inside the cluster |
| **NodePort** | Exposes the app on a port of every node (30000-32767) |
| **LoadBalancer** | Creates a cloud load balancer (e.g., AWS ELB/NLB) |
| **ExternalName** | Maps a Service to an external DNS name |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
```

### 4.5 Namespace
- A **virtual partition** inside a cluster
- Used to separate environments or teams (`dev`, `staging`, `prod`)
- Supports resource quotas and access control per namespace
- Default namespaces: `default`, `kube-system`, `kube-public`, `kube-node-lease`

### 4.6 Labels, Selectors, and Annotations
- **Labels:** key-value tags on objects (`app: my-app`)
- **Selectors:** find objects using labels (how a Service finds its Pods)
- **Annotations:** non-identifying metadata (descriptions, tool settings)

---

## 5. Configuration and Storage

| Object | Purpose |
|---|---|
| **ConfigMap** | Non-sensitive configuration (URLs, feature flags) |
| **Secret** | Sensitive data (passwords, tokens). Base64 **encoded**, not encrypted by default |
| **Volume** | Storage attached to a Pod (lives as long as the Pod) |
| **PersistentVolume (PV)** | A piece of storage provisioned in the cluster |
| **PersistentVolumeClaim (PVC)** | A Pod's request for storage |
| **StorageClass** | Defines types of storage for dynamic provisioning |

---

## 6. Other Important Objects

| Object | Purpose |
|---|---|
| **Ingress** | Routes external HTTP/HTTPS traffic to Services by host or path |
| **StatefulSet** | For stateful apps (databases): stable names and storage |
| **DaemonSet** | Runs one Pod on **every node** (log/monitoring agents) |
| **Job** | Runs a task once until completion |
| **CronJob** | Runs Jobs on a schedule |
| **HPA** | Horizontal Pod Autoscaler: scales Pods by CPU/memory |
| **ResourceQuota** | Limits total resources in a namespace |
| **RBAC** | Role-based access control (who can do what) |

---

## 7. How It Works End to End

1. You write a Deployment YAML and run `kubectl apply -f deployment.yaml`
2. The **API Server** validates it and stores it in **etcd**
3. The **Controller Manager** creates the ReplicaSet and Pods
4. The **Scheduler** assigns each Pod to a suitable node
5. The **kubelet** on that node pulls the image and starts the containers
6. A **Service** exposes the Pods with a stable address
7. Controllers keep checking: *"Is actual state equal to desired state?"* If not, they fix it

---

## 8. Hands-On: Your First Deployment

> **Prerequisites:** a local cluster (Minikube, kind, or Docker Desktop) or a cloud cluster, and `kubectl` installed.

**Step 1: Verify the cluster**
```bash
kubectl cluster-info
kubectl get nodes
```
<!-- 📸 Screenshot: output of kubectl get nodes -->

**Step 2: Create a namespace**
```bash
kubectl create namespace demo
```

**Step 3: Create the Deployment**
```bash
kubectl create deployment web --image=nginx:1.25 --replicas=3 -n demo
kubectl get pods -n demo
```
<!-- 📸 Screenshot: 3 pods in Running state -->

**Step 4: Expose it with a Service**
```bash
kubectl expose deployment web --port=80 --type=NodePort -n demo
kubectl get svc -n demo
```
<!-- 📸 Screenshot: service with NodePort -->

**Step 5: Scale**
```bash
kubectl scale deployment web --replicas=5 -n demo
```

**Step 6: Rolling update and rollback**
```bash
kubectl set image deployment/web nginx=nginx:1.26 -n demo
kubectl rollout status deployment/web -n demo
kubectl rollout undo deployment/web -n demo
```
<!-- 📸 Screenshot: rollout status -->

**Step 7: Clean up**
```bash
kubectl delete namespace demo
```

---

## 9. Essential kubectl Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes -o wide

# View resources
kubectl get pods -n <namespace>
kubectl get all -A
kubectl describe pod <pod-name>

# Logs and debugging
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container>      # multi-container pod
kubectl exec -it <pod-name> -- /bin/sh
kubectl get events --sort-by=.metadata.creationTimestamp

# Create / update / delete
kubectl apply -f file.yaml
kubectl delete -f file.yaml

# Scaling and updates
kubectl scale deployment <name> --replicas=5
kubectl rollout status deployment <name>
kubectl rollout history deployment <name>
kubectl rollout undo deployment <name>

# Namespaces and context
kubectl create namespace dev
kubectl config get-contexts
kubectl config set-context --current --namespace=dev

# Port forwarding for quick tests
kubectl port-forward svc/<service-name> 8080:80
```

---

## 10. Interview Questions and Answers

### Section A: Basics

**Q1. What is Kubernetes?**
An open-source container orchestration platform that automates deployment, scaling, networking, and management of containerized applications.

**Q2. Why do we need Kubernetes when Docker already runs containers?**
Docker runs containers on a single host. Kubernetes manages containers across many hosts and adds self-healing, auto-scaling, load balancing, rolling updates, and service discovery.

**Q3. What is a cluster?**
A set of machines (nodes) working together to run containerized applications. It consists of a control plane and worker nodes.

**Q4. What is a node?**
A single machine (physical or virtual) in the cluster. Worker nodes run Pods; control plane nodes run management components.

**Q5. What is a Pod?**
The smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network namespace and storage volumes.

**Q6. Can a Pod have multiple containers? When?**
Yes. Used for tightly coupled helpers, such as a **sidecar** (log shipper), **init container** (setup tasks), or **adapter/ambassador** patterns.

**Q7. Why are Pods considered ephemeral?**
Pods can be killed, rescheduled, or replaced at any time, and each new Pod gets a new IP. That is why Services exist.

**Q8. What is a namespace and why use it?**
A virtual cluster inside a cluster. It isolates resources by team or environment and allows quotas and RBAC per namespace.

**Q9. What are the default namespaces?**
`default`, `kube-system` (system components), `kube-public`, and `kube-node-lease` (node heartbeats).

**Q10. What are labels and selectors?**
Labels are key-value pairs attached to objects. Selectors filter objects by labels. Services, ReplicaSets, and Deployments use selectors to find their Pods.

### Section B: Architecture

**Q11. Explain the Kubernetes architecture.**
The **control plane** (API Server, etcd, Scheduler, Controller Manager) makes decisions. **Worker nodes** (kubelet, kube-proxy, container runtime) run the workloads.

**Q12. What does the API Server do?**
It is the central entry point. It validates and processes REST requests, and is the only component that talks directly to etcd.

**Q13. What is etcd?**
A distributed, consistent key-value store that holds all cluster data. Backing it up is critical for disaster recovery.

**Q14. What does the Scheduler do?**
It watches for unscheduled Pods and assigns them to nodes based on resources, taints/tolerations, affinity rules, and constraints.

**Q15. What does the Controller Manager do?**
It runs controllers (Node, ReplicaSet, Deployment, etc.) that continuously compare actual state with desired state and correct differences.

**Q16. What does the kubelet do?**
An agent on every node that talks to the API Server, starts the Pod containers via the runtime, and reports their health.

**Q17. What does kube-proxy do?**
Maintains network rules (iptables/IPVS) on each node so traffic to a Service reaches the right Pods.

**Q18. What is a container runtime?**
Software that runs containers, such as containerd or CRI-O. Kubernetes talks to it via the **CRI** (Container Runtime Interface).

**Q19. What happens when you run `kubectl apply -f deployment.yaml`?**
kubectl sends the request to the API Server, which authenticates, validates, and stores it in etcd. Controllers create the ReplicaSet and Pods, the Scheduler assigns nodes, and kubelets start the containers.

**Q20. Is the control plane involved when an app is running?**
Not for serving traffic. If the control plane goes down, existing Pods keep running, but you cannot make changes, and failed Pods will not be replaced.

### Section C: Workloads

**Q21. What is a ReplicaSet?**
It ensures a set number of identical Pods are running at all times.

**Q22. What is the difference between a ReplicaSet and a Deployment?**
A Deployment manages ReplicaSets and adds rolling updates, rollback, and version history. A ReplicaSet alone only maintains the Pod count.

**Q23. What are the Deployment update strategies?**
**RollingUpdate** (default; replaces Pods gradually, controlled by `maxSurge` and `maxUnavailable`) and **Recreate** (deletes all old Pods, then creates new ones, causing downtime).

**Q24. How do you roll back a Deployment?**
`kubectl rollout undo deployment <name>` (or `--to-revision=<n>`). Check history with `kubectl rollout history`.

**Q25. What is a StatefulSet and how is it different from a Deployment?**
A StatefulSet is for stateful apps. It provides stable Pod names (`db-0`, `db-1`), stable network identity, ordered startup, and dedicated persistent storage per Pod.

**Q26. What is a DaemonSet?**
Ensures one Pod runs on every (or selected) node. Used for log collectors, monitoring agents, and network plugins.

**Q27. What is the difference between a Job and a CronJob?**
A Job runs a task to completion once. A CronJob creates Jobs on a schedule (cron syntax).

**Q28. What is an init container?**
A container that runs and must complete **before** the main containers start. Used for setup, waiting for dependencies, or migrations.

### Section D: Networking

**Q29. What is a Service and why is it needed?**
It gives Pods a stable IP and DNS name and load-balances traffic across them, even as Pods are replaced.

**Q30. Explain Service types.**
**ClusterIP** (internal only), **NodePort** (exposes on each node's port), **LoadBalancer** (cloud load balancer), **ExternalName** (DNS alias).

**Q31. What is the difference between `port`, `targetPort`, and `nodePort`?**
`port` is the Service's port, `targetPort` is the container's port, and `nodePort` is the port opened on each node (NodePort type only).

**Q32. What is Ingress?**
An object that defines HTTP/HTTPS routing rules (host and path based) to Services. It requires an **Ingress Controller** (NGINX, AWS ALB controller, etc.) to work.

**Q33. Difference between Ingress and LoadBalancer Service?**
A LoadBalancer Service creates one cloud load balancer per Service (costly). Ingress uses one entry point to route to many Services with Layer 7 rules, TLS, and path routing.

**Q34. What is a Headless Service?**
A Service with `clusterIP: None`. It returns the individual Pod IPs through DNS instead of a single virtual IP. Used with StatefulSets.

**Q35. How do Pods communicate with each other?**
Every Pod gets its own IP, and all Pods can reach each other without NAT. This is implemented by a CNI plugin (Calico, Flannel, Cilium, AWS VPC CNI).

**Q36. What is a NetworkPolicy?**
Rules that control which Pods can talk to which. By default all traffic is allowed. It needs a CNI that supports it.

### Section E: Configuration and Storage

**Q37. ConfigMap vs Secret?**
ConfigMap holds non-sensitive config. Secret holds sensitive data. Secrets are base64 encoded, **not encrypted** by default, so enable encryption at rest and RBAC.

**Q38. How can a Pod consume a ConfigMap or Secret?**
As environment variables, as command-line args, or as files mounted via a volume.

**Q39. What is the difference between a Volume and a PersistentVolume?**
A regular volume's lifetime is tied to the Pod. A PV is independent cluster storage that survives Pod deletion.

**Q40. What are PV, PVC, and StorageClass?**
PV is the actual storage, PVC is a request for storage by a user, and StorageClass enables dynamic provisioning of PVs.

### Section F: Scaling, Scheduling, and Reliability

**Q41. What is HPA?**
Horizontal Pod Autoscaler: automatically changes the number of Pods based on metrics like CPU, memory, or custom metrics. Requires Metrics Server.

**Q42. Difference between HPA, VPA, and Cluster Autoscaler?**
HPA changes Pod count. VPA adjusts Pod CPU/memory requests. Cluster Autoscaler adds or removes nodes.

**Q43. What are resource requests and limits?**
**Requests** are the guaranteed minimum used for scheduling. **Limits** are the maximum allowed. Exceeding the memory limit causes an OOMKill; exceeding CPU leads to throttling.

**Q44. What are liveness, readiness, and startup probes?**
**Liveness:** is the container alive? If not, restart it. **Readiness:** is it ready for traffic? If not, remove it from Service endpoints. **Startup:** for slow-starting apps, delays the other probes.

**Q45. What are taints and tolerations?**
Taints on nodes repel Pods. Tolerations on Pods allow them to be scheduled onto tainted nodes.

**Q46. What is node affinity vs nodeSelector?**
`nodeSelector` is a simple label match. Node affinity is more expressive, supporting required and preferred rules and operators.

### Section G: Security and Troubleshooting

**Q47. What is RBAC?**
Role-Based Access Control. **Role/ClusterRole** define permissions; **RoleBinding/ClusterRoleBinding** assign them to users, groups, or ServiceAccounts.

**Q48. A Pod is stuck in `Pending`. What do you check?**
`kubectl describe pod`. Common causes: insufficient CPU/memory, no node matching selectors or taints, unbound PVC, or image pull secrets missing.

**Q49. A Pod shows `CrashLoopBackOff`. How do you troubleshoot?**
Check `kubectl logs <pod> --previous`, `kubectl describe pod` (exit codes, events), the command/args, missing ConfigMap/Secret, failing probes, or OOMKilled status.

**Q50. A Pod shows `ImagePullBackOff`. What are the causes?**
Wrong image name or tag, private registry without `imagePullSecrets`, network issues, or registry rate limits.

### Bonus: Scenario Questions

**S1. A Service is not reachable. What is your approach?**
1. Check Pods are `Running` and `Ready`
2. Check Service selector matches Pod labels (`kubectl describe svc`)
3. Check endpoints: `kubectl get endpoints <svc>`
4. Test from inside the cluster with a temporary Pod
5. Check NetworkPolicies, Ingress rules, and security groups

**S2. How do you achieve zero-downtime deployments?**
Use RollingUpdate with proper `maxUnavailable`, readiness probes, multiple replicas, a PodDisruptionBudget, and graceful shutdown handling.

**S3. How would you separate dev, staging, and prod?**
Separate namespaces (or separate clusters for prod), with ResourceQuotas, RBAC, and NetworkPolicies per environment.

**S4. How do you keep secrets secure in Kubernetes?**
Enable etcd encryption at rest, restrict RBAC, use external secret managers (AWS Secrets Manager, Vault) with the External Secrets Operator, and never commit Secrets to Git.

---

## 11. Quick Revision Table

| Term | One-line meaning |
|---|---|
| **Cluster** | Group of machines running your containers |
| **Node** | A single machine in the cluster |
| **Pod** | Smallest unit; wraps containers |
| **ReplicaSet** | Keeps N Pods running |
| **Deployment** | Manages ReplicaSets; scaling, updates, rollback |
| **StatefulSet** | Stateful apps with stable identity and storage |
| **DaemonSet** | One Pod per node |
| **Service** | Stable endpoint to reach Pods |
| **Ingress** | HTTP/HTTPS routing from outside |
| **Namespace** | Virtual separation inside a cluster |
| **ConfigMap / Secret** | Configuration / sensitive data |
| **PV / PVC** | Persistent storage and the request for it |
| **HPA** | Autoscale Pods by metrics |
| **RBAC** | Access control |

### Easy Way to Remember

> **Cluster** = the whole company
> **Node** = an office building
> **Namespace** = a department
> **Pod** = an employee at a desk
> **Deployment** = the HR policy "always keep 3 employees on this team"
> **Service** = the reception desk that routes visitors to the right employee

---

## Next Steps

- Practice on **Minikube** or **kind**
- Try managed Kubernetes: **Amazon EKS**
- Learn **Helm**, **Ingress controllers**, and **ArgoCD** (GitOps)
- Prepare for the **CKA / CKAD** certifications

---

⭐ If this guide helped you, consider giving the repo a star!