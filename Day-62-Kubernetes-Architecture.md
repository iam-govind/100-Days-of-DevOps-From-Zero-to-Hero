# Day 62/100 — Kubernetes Architecture

## Learning objectives

By the end of this lesson, you should be able to:

- Explain the purpose of Kubernetes and container orchestration.
- Distinguish the control plane from worker nodes.
- Describe the main Kubernetes components.
- Explain what happens after `kubectl apply`.
- Inspect a Kubernetes cluster using `kubectl`.
- Identify common architecture-related troubleshooting areas.

## 1. What is Kubernetes?

Kubernetes (often called K8s) is an open-source container orchestration platform. It automates the deployment, scaling, networking, and recovery of containerized applications.

Running containers manually is manageable for a small application. As the number of services and containers grows, you need automation for:

- Scheduling workloads onto machines.
- Restarting failed containers and replacing failed Pods.
- Scaling applications.
- Service discovery and networking.
- Rolling out application updates.
- Maintaining the desired number of application replicas.

Kubernetes provides these capabilities through a cluster.

## 2. Kubernetes cluster architecture

A Kubernetes cluster consists of two broad parts:

- **Control plane:** Manages the cluster and makes decisions.
- **Worker nodes:** Run application workloads inside Pods.

```text
                         kubectl
                            |
                            v
                 +----------------------+
                 |     CONTROL PLANE    |
                 |                      |
                 |  kube-apiserver      |
                 |  etcd                |
                 |  kube-scheduler      |
                 |  kube-controller-mgr |
                 |  cloud-controller*  |
                 +----------------------+
                            |
                    schedules workloads
                            |
               +------------+------------+
               |                         |
               v                         v
        +-------------+           +-------------+
        |  WORKER 1   |           |  WORKER 2   |
        |             |           |             |
        |  kubelet    |           |  kubelet    |
        |  runtime    |           |  runtime    |
        |  Pods       |           |  Pods       |
        +-------------+           +-------------+

* cloud-controller-manager is used when applicable.
```

## 3. Control plane components

The control plane manages the cluster's overall state.

### 3.1 kube-apiserver

The API server is the front door to the Kubernetes control plane.

- Exposes the Kubernetes API.
- Receives requests from `kubectl`, controllers, and other clients.
- Performs authentication, authorization, and admission checks.
- Coordinates persistence of accepted cluster state in etcd.

Think of it as the communication hub of the cluster.

### 3.2 etcd

etcd is a consistent, distributed key-value store used to hold Kubernetes cluster data.

It stores information such as:

- Cluster configuration and state.
- Kubernetes objects and their metadata.
- The desired state declared for workloads.

Important: Protect etcd carefully. Backups and secure access are essential for cluster recovery.

### 3.3 kube-scheduler

The scheduler watches for newly created Pods that do not yet have a node assigned.

It evaluates factors such as:

- Available CPU and memory.
- Resource requests.
- Node selectors and affinity rules.
- Taints and tolerations.
- Other scheduling constraints.

It selects a suitable node. The kubelet on that node then takes responsibility for making the Pod run.

### 3.4 kube-controller-manager

This component runs controllers. Controllers continuously compare the desired state with the observed state and take action to reduce differences.

Example: If a Deployment requests three replicas but only two are available, the relevant controllers work to create the missing Pod.

### 3.5 cloud-controller-manager

In cloud-integrated clusters, this component connects Kubernetes to the cloud provider's APIs. Depending on the provider and configuration, it can manage cloud-related node information, routes, and load balancers.

It is not used in every Kubernetes environment.

## 4. Worker node components

Worker nodes provide the compute capacity for application workloads.

### 4.1 kubelet

The kubelet is the node agent. It receives Pod specifications for its node and works with the container runtime to ensure the specified containers are running and healthy.

### 4.2 Container runtime

The container runtime runs containers. Common examples include:

- containerd
- CRI-O

Kubernetes communicates with runtimes through the Container Runtime Interface (CRI).

### 4.3 kube-proxy and cluster networking

In clusters that use kube-proxy, it maintains network rules that help implement Kubernetes Service networking.

Some clusters use alternative networking data planes that do not require kube-proxy. Pod networking is also provided by the cluster's network plugin (CNI implementation).

### 4.4 Pods

A Pod is the smallest deployable unit in Kubernetes. It contains one or more closely related containers that share network and storage resources as configured.

A Pod is not the same thing as a node or a container:

- **Node:** A machine (physical or virtual) in the cluster.
- **Pod:** A Kubernetes workload unit scheduled onto a node.
- **Container:** A running application process packaged with its dependencies.

## 5. What happens when you deploy an application?

Suppose you run:

```bash
kubectl apply -f deployment.yaml
```

A simplified flow is:

1. `kubectl` sends the request to the API server.
2. The API server authenticates and authorizes the request and runs applicable admission checks.
3. The accepted object state is persisted in etcd.
4. Controllers observe that the desired number of Pods needs to exist.
5. The scheduler assigns unscheduled Pods to suitable worker nodes.
6. The kubelet on each selected node observes the assigned Pod specification.
7. The kubelet asks the container runtime to pull images if necessary and start containers.
8. The cluster networking implementation enables Pod communication and Services provide stable access to workloads.
9. Controllers and node agents continue monitoring and reconciling state.

This is a simplified learning model. In practice, controllers, the scheduler, kubelets, and other components work asynchronously through the API.

## 6. Desired state vs actual state

Kubernetes is declarative: you describe the state you want, and Kubernetes works to make the cluster match it.

Example:

- Desired state: three replicas.
- Actual state: two healthy Pods.
- Reconciliation: controllers work to create a replacement Pod.

The goal is to keep actual state aligned with desired state, not merely to execute a command once.

## 7. Managed Kubernetes

Services such as Azure Kubernetes Service (AKS), Amazon Elastic Kubernetes Service (EKS), and Google Kubernetes Engine (GKE) provide managed Kubernetes.

The cloud provider manages important parts of the control plane, while you still manage workloads and many cluster configuration responsibilities. The exact division of responsibilities depends on the service and configuration.

On managed clusters, you may not be able to inspect control-plane components as ordinary Pods in `kube-system`.

## 8. Hands-on lab: inspect a local cluster

This lab uses Minikube with Docker Desktop as the driver. Install Docker Desktop, `kubectl`, and Minikube first. Do not run the start command until Docker Desktop is running.

### Step 1 — Check kubectl

```bash
kubectl version --client
```

This confirms that the Kubernetes command-line client is installed.

### Step 2 — Start Minikube

```bash
minikube start --driver=docker
```

Minikube creates a local Kubernetes cluster using Docker.

### Step 3 — Check cluster information

```bash
kubectl cluster-info
minikube status
```

### Step 4 — List nodes

```bash
kubectl get nodes -o wide
```

You should see your local node, commonly named `minikube`, with a `Ready` status after startup completes.

### Step 5 — List Pods across namespaces

```bash
kubectl get pods -A
```

This lists Pods in all namespaces. System Pods commonly appear in the `kube-system` namespace.

### Step 6 — Inspect system Pods

```bash
kubectl get pods -n kube-system
```

Look at the names and statuses. The exact list depends on the Minikube version and configuration.

### Step 7 — Describe the node

```bash
kubectl describe node minikube
```

If your node has a different name, use the name shown by `kubectl get nodes`.

Review capacity, allocatable resources, conditions, labels, and running Pods.

### Step 8 — Check API server readiness

```bash
kubectl get --raw='/readyz?verbose'
```

This queries the API server's readiness endpoint. Access and output can vary by cluster configuration and permissions.

### Step 9 — Stop the local cluster

```bash
minikube stop
```

Alternative: If you already use kind, you can create and inspect a local cluster with kind instead. Avoid running both setups unless you specifically want to compare them.

## 9. Common troubleshooting

| Symptom | Possible cause | What to check |
|---|---|---|
| Minikube cannot start | Docker Desktop is stopped or driver issue | Start Docker Desktop; inspect `minikube status` |
| Node is `NotReady` | Runtime, networking, or resource issue | `kubectl describe node <node-name>` |
| Pod is `Pending` | Insufficient resources or scheduling constraints | `kubectl describe pod <pod-name> -n <namespace>` |
| `ImagePullBackOff` | Image name, tag, registry access, or credentials | `kubectl describe pod <pod-name> -n <namespace>` |
| kubectl connects to wrong cluster | Incorrect kubeconfig context | `kubectl config get-contexts` and `kubectl config current-context` |
| System Pods are failing | Cluster component, networking, or resource issue | `kubectl get pods -n kube-system` and describe the affected Pod |

Useful commands:

```bash
kubectl config current-context
kubectl config get-contexts
kubectl get nodes
kubectl get pods -A
kubectl describe node <node-name>
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

## 10. Production best practices

- Use a highly available control plane for production-critical clusters.
- Back up etcd where you operate and manage it; understand the provider's backup and recovery model for managed clusters.
- Apply least-privilege RBAC permissions.
- Use network policies where supported by your network plugin.
- Set appropriate CPU and memory requests and limits.
- Monitor cluster, node, and application health.
- Plan and test Kubernetes upgrades.
- Store secrets securely and restrict access.
- Define resource quotas and limit ranges where appropriate.
- Test backup restoration and disaster-recovery procedures.

## 11. Interview questions

1. What is Kubernetes, and why do we need container orchestration?
2. What is the difference between the control plane and a worker node?
3. What is the responsibility of kube-apiserver?
4. What information does etcd store?
5. How does kube-scheduler select a node?
6. What does kubelet do?
7. What is the difference between a Pod, container, and node?
8. What is reconciliation in Kubernetes?
9. Is kube-proxy mandatory in every Kubernetes cluster?
10. What responsibilities remain with a team using managed Kubernetes such as AKS?

## 12. Practical challenge

Using your local cluster:

1. Start Minikube.
2. Capture the output of `kubectl get nodes -o wide`.
3. List all Pods using `kubectl get pods -A`.
4. Inspect the `kube-system` namespace.
5. Describe the Minikube node.
6. Write down which components are visible to you and which control-plane details are managed or hidden by the local cluster setup.
7. Stop Minikube after completing the lab.

### Suggested screenshots for GitHub

- `kubectl get nodes -o wide`
- `kubectl get pods -A`
- `kubectl get pods -n kube-system`
- `kubectl describe node minikube`

## 13. Key takeaways

- The control plane makes cluster-management decisions.
- Worker nodes run application Pods.
- The API server is the main interface to Kubernetes.
- etcd stores cluster state.
- The scheduler assigns Pods to nodes.
- kubelet works with the container runtime to run Pods.
- Controllers continuously reconcile actual state with desired state.

## 14. Official documentation

- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Kubernetes Architecture Concepts](https://kubernetes.io/docs/concepts/architecture/)
- [Minikube — Get Started](https://minikube.sigs.k8s.io/docs/start/)

## Coming next

**Day 63 — Kubernetes Pods**

We will learn what Pods are, why Kubernetes uses them, how to create and inspect Pods, and how to troubleshoot common Pod states.
