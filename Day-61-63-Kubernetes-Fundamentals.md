# Days 61–63/100 — Kubernetes Fundamentals

**Combined topics**
- Day 61 — Introduction to Kubernetes
- Day 62 — Kubernetes Architecture
- Day 63 — Kubernetes Pods

This combined lesson catches up on Days 61 and 62 while completing Day 63. The original roadmap remains unchanged: Day 64 is Deployments.

---

# Day 61 — Introduction to Kubernetes

## 1. What is Kubernetes?

Kubernetes (often shortened to K8s) is an open-source container orchestration platform. It helps automate deployment, scaling, networking, and recovery of containerized applications.

A container packages an application and its dependencies. Running a few containers manually may be straightforward, but operating many containers across multiple machines introduces challenges:

- What happens when a container or machine fails?
- How do you run multiple copies of an application?
- How do you scale when traffic increases?
- How do services discover and communicate with one another?
- How do you release a new version safely?
- How do you keep the application close to the state you requested?

Kubernetes provides APIs and controllers that help manage these challenges.

## 2. Why do we need Kubernetes?

Imagine an application running in three containers on one server. If one container stops, an operator may need to restart it manually. If traffic increases, the operator may need to add more containers and configure traffic distribution.

Kubernetes automates many of these operational tasks.

### Key capabilities

1. **Self-healing:** When a Pod managed by a controller fails, Kubernetes works to replace it and maintain the requested number of replicas.
2. **Scaling:** You can increase or decrease the number of application replicas. Horizontal Pod Autoscaling can adjust replicas based on configured metrics.
3. **Service discovery and load balancing:** Kubernetes Services provide stable network endpoints for a changing set of Pods.
4. **Rolling updates and rollbacks:** Deployments support controlled application updates and rollbacks.
5. **Scheduling:** Kubernetes places Pods on nodes based on resource requests and scheduling constraints.
6. **Configuration and secrets:** Applications can consume configuration through ConfigMaps and sensitive values through Secrets, with appropriate security controls.
7. **Resource management:** You can define CPU and memory requests and limits for containers.

Kubernetes does not automatically guarantee application high availability. Good application design, multiple replicas, suitable cluster capacity, health checks, networking, and failure-domain planning are also important.

## 3. Basic Kubernetes terminology

| Term | Meaning |
|---|---|
| Cluster | A set of machines managed by Kubernetes |
| Node | A machine (physical or virtual) in a cluster |
| Pod | The smallest deployable Kubernetes workload unit |
| Container | A running application process packaged with its dependencies |
| Deployment | Manages replicated Pods and declarative application updates |
| Replica | A desired copy of an application Pod |
| Service | Provides a stable network endpoint for a group of Pods |
| Namespace | Organizes resources within a cluster |
| ConfigMap | Stores non-sensitive configuration data |
| Secret | Holds sensitive data, subject to appropriate access and encryption controls |

## 4. Docker vs Kubernetes

Docker tooling is commonly used to build and run containers. Kubernetes orchestrates containerized workloads across a cluster.

| Docker/container tooling | Kubernetes |
|---|---|
| Builds and packages container images | Schedules and manages workloads |
| Runs containers using a container runtime | Manages Pods across cluster nodes |
| Docker Compose can manage multi-container applications on a host | Kubernetes manages application workloads across a cluster |
| Focuses on container build and runtime workflows | Focuses on orchestration and desired-state management |

They are complementary, not direct replacements. Kubernetes does not require Docker Engine as its container runtime. Common Kubernetes runtimes include containerd and CRI-O.

## 5. First look at Kubernetes architecture

A cluster has two broad parts:

- **Control plane:** Exposes the Kubernetes API and makes cluster-management decisions.
- **Worker nodes:** Provide compute capacity and run application Pods.

```text
                  Kubernetes Cluster
                          |
             +------------+------------+
             |                         |
             v                         v
       CONTROL PLANE               WORKER NODE
       - API Server                - kubelet
       - Scheduler                 - Container runtime
       - Controllers               - Pods
       - etcd
```

We will explore the components in detail in Day 62.

---

# Day 62 — Kubernetes Architecture

## 1. High-level architecture

A Kubernetes cluster consists of a control plane and one or more worker nodes.

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

## 2. Control plane components

The control plane manages the cluster's overall state and makes decisions about workloads.

### 2.1 kube-apiserver

The API server is the front door to the Kubernetes control plane.

- Exposes the Kubernetes API.
- Receives requests from `kubectl`, controllers, and other clients.
- Performs authentication, authorization, and admission checks.
- Coordinates persistence of accepted cluster state in etcd.

Think of it as the communication hub of the cluster.

### 2.2 etcd

etcd is a consistent, distributed key-value store used to hold Kubernetes cluster data.

It stores information such as:

- Cluster configuration and state.
- Kubernetes objects and their metadata.
- The desired state declared for workloads.

Protect etcd carefully. Backups and secure access are essential for cluster recovery.

### 2.3 kube-scheduler

The scheduler watches for newly created Pods that do not yet have a node assigned.

It evaluates factors such as:

- Available CPU and memory.
- Resource requests.
- Node selectors and affinity rules.
- Taints and tolerations.
- Other scheduling constraints.

It selects a suitable node. The kubelet on that node then takes responsibility for making the Pod run.

### 2.4 kube-controller-manager

This component runs controllers. Controllers continuously compare the desired state with the observed state and take action to reduce differences.

Example: If a Deployment requests three replicas but only two are available, the relevant controllers work to create the missing Pod.

### 2.5 cloud-controller-manager

In cloud-integrated clusters, this component connects Kubernetes to the cloud provider's APIs. Depending on the provider and configuration, it can manage cloud-related node information, routes, and load balancers.

It is not used in every Kubernetes environment.

## 3. Worker node components

Worker nodes provide the compute capacity for application workloads.

### 3.1 kubelet

The kubelet is the node agent. It receives Pod specifications for its node and works with the container runtime to ensure the specified containers are running and healthy.

### 3.2 Container runtime

The container runtime runs containers. Common examples include:

- containerd
- CRI-O

Kubernetes communicates with runtimes through the Container Runtime Interface (CRI).

### 3.3 kube-proxy and cluster networking

In clusters that use kube-proxy, it maintains network rules that help implement Kubernetes Service networking.

Some clusters use alternative networking data planes that do not require kube-proxy. Pod networking is also provided by the cluster's network plugin (CNI implementation).

### 3.4 Pods

A Pod is the smallest deployable unit in Kubernetes. It contains one or more closely related containers that share network and storage resources as configured.

A Pod is not the same thing as a node or a container:

- **Node:** A machine (physical or virtual) in the cluster.
- **Pod:** A Kubernetes workload unit scheduled onto a node.
- **Container:** A running application process packaged with its dependencies.

## 4. What happens when you deploy an application?

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

## 5. Desired state vs actual state

Kubernetes is declarative: you describe the state you want, and Kubernetes works to make the cluster match it.

Example:

- Desired state: three replicas.
- Actual state: two healthy Pods.
- Reconciliation: controllers work to create a replacement Pod.

The goal is to keep actual state aligned with desired state, not merely to execute a command once.

## 6. Managed Kubernetes

Services such as Azure Kubernetes Service (AKS), Amazon Elastic Kubernetes Service (EKS), and Google Kubernetes Engine (GKE) provide managed Kubernetes.

The cloud provider manages important parts of the control plane, while you still manage workloads and many cluster configuration responsibilities. The exact division of responsibilities depends on the service and configuration.

On managed clusters, you may not be able to inspect control-plane components as ordinary Pods in `kube-system`.

---

# Day 63 — Kubernetes Pods

## 1. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes. Kubernetes schedules Pods onto worker nodes, and each Pod contains one or more containers that are tightly coupled.

Most application Pods contain one main application container. Some Pods contain additional helper containers, often called sidecars.

Containers in the same Pod:

- Share the same network namespace and IP address.
- Can communicate with each other through `localhost`.
- Can share storage volumes when configured.
- Are scheduled together on the same node.
- Have separate container filesystems unless storage is shared or otherwise mounted.

A Pod is not a permanent machine. It can be replaced, and a replacement Pod can have a different IP address.

## 2. Pod vs container vs node

| Item | Meaning | Example |
|---|---|---|
| Node | Machine in the cluster | Worker VM |
| Pod | Kubernetes deployable unit | `nginx-pod` |
| Container | Runs the application process | NGINX container |

```text
             Kubernetes Cluster
                     |
                Worker Node
                     |
                 +-------+
                 |  Pod  |
                 |       |
                 | +---+ |
                 | |App| |
                 | +---+ |
                 +-------+
```

## 3. Create your first Pod using kubectl

Make sure your local cluster is running:

```bash
minikube start --driver=docker
```

Create a Pod:

```bash
kubectl run nginx-pod --image=nginx:1.27
```

Check the Pod:

```bash
kubectl get pods
```

Wait until the Pod shows `Running` and `1/1` ready.

Get more details:

```bash
kubectl get pods -o wide
kubectl describe pod nginx-pod
```

The `describe` command is particularly useful when a Pod is not starting. Review its Events section.

## 4. Create a Pod using a YAML manifest

In real DevOps workflows, declarative YAML manifests are commonly used so configuration can be reviewed, version-controlled, and applied consistently.

Create a file named `nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-yaml
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

Apply the manifest:

```bash
kubectl apply -f nginx-pod.yaml
```

Verify:

```bash
kubectl get pod nginx-pod-yaml
kubectl describe pod nginx-pod-yaml
```

Check its labels:

```bash
kubectl get pods --show-labels
```

## 5. Understand the important YAML fields

| Field | Purpose |
|---|---|
| `apiVersion` | API version for the resource |
| `kind` | Type of Kubernetes object |
| `metadata.name` | Name of the Pod |
| `metadata.labels` | Key-value labels used to organize and select resources |
| `spec` | Desired specification |
| `containers` | List of containers in the Pod |
| `name` | Container name |
| `image` | Container image to run |
| `ports` | Documents ports the container listens on; it does not by itself publish the Pod externally |

## 6. Common Pod phases and states

The Pod phase is a high-level summary. `kubectl get pods` also displays a STATUS column that can show more specific information about container startup and failure.

| Phase / displayed status | Meaning |
|---|---|
| Pending | Pod has been accepted but one or more containers are not yet running; it may be waiting for scheduling or setup |
| Running | Pod is bound to a node and at least one container is running or starting/restarting |
| Succeeded | All containers terminated successfully and will not be restarted |
| Failed | All containers have terminated and at least one failed |
| Unknown | Kubernetes cannot determine the Pod state, often due to node communication problems |
| `CrashLoopBackOff` | A container repeatedly crashes and Kubernetes is delaying restart attempts |
| `ImagePullBackOff` | Kubernetes cannot pull the requested image and is backing off before retrying |
| `ErrImagePull` | An image pull attempt failed |

`CrashLoopBackOff` and `ImagePullBackOff` are displayed statuses, not Pod phases.

## 7. Inspect Pod logs

```bash
kubectl logs nginx-pod
```

If a Pod has multiple containers, specify the container:

```bash
kubectl logs <pod-name> -c <container-name>
```

For logs from the previous instance of a restarted container:

```bash
kubectl logs <pod-name> --previous
```

## 8. Execute a command inside a Pod

For an interactive shell in an image that contains a shell:

```bash
kubectl exec -it nginx-pod -- /bin/sh
```

Inside the container, try:

```sh
hostname
```

Exit the shell:

```sh
exit
```

Not every container image includes `/bin/sh` or debugging utilities. A minimal or distroless image may require a different debugging approach.

## 9. Access a Pod locally with port-forwarding

Forward local port 8080 to port 80 in the Pod:

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

Keep the command running. In another terminal:

```bash
curl http://127.0.0.1:8080
```

You should receive the NGINX welcome page HTML. Stop port-forwarding with `Ctrl+C`.

Port-forwarding is useful for local testing. It is not a production exposure strategy.

## 10. Delete a Pod

Delete a Pod created directly:

```bash
kubectl delete pod nginx-pod
```

Delete the YAML-created Pod:

```bash
kubectl delete -f nginx-pod.yaml
```

A Pod created directly with `kubectl run` is not automatically recreated after deletion. A Pod managed by a Deployment or another controller can be recreated to maintain the desired state.

## 11. Pod resource requests and limits

In production, define resource requirements so the scheduler can make informed placement decisions and containers have appropriate resource boundaries.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resources
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      resources:
        requests:
          cpu: "100m"
          memory: "64Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
```

- **Request:** Amount used for scheduling decisions and resource reservation.
- **Limit:** Maximum amount a container is allowed to use for that resource, subject to Kubernetes resource behavior.

CPU is throttled when its limit is reached. A container that exceeds its memory limit can be terminated with an out-of-memory (OOM) condition.

Apply and inspect:

```bash
kubectl apply -f nginx-resources.yaml
kubectl describe pod nginx-resources
```

## 12. Common Pod troubleshooting

| Symptom | Possible cause | First checks |
|---|---|---|
| Pending | Insufficient resources, scheduling constraints, or volume setup issue | `kubectl describe pod <pod-name>` |
| ImagePullBackOff | Wrong image/tag, registry access, or missing credentials | `kubectl describe pod <pod-name>` |
| CrashLoopBackOff | Application exits, bad configuration, or failed dependency | `kubectl logs <pod-name>` and `kubectl logs <pod-name> --previous` |
| OOMKilled | Container exceeded memory limit or node memory pressure | `kubectl describe pod <pod-name>` |
| Unknown | Node or kubelet communication issue | `kubectl get nodes` and `kubectl describe node <node-name>` |
| Pod is Running but app is unreachable | Application not listening, wrong port, or networking issue | Check logs, ports, and port-forward |

Useful diagnostic commands:

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get nodes
```

## 13. Production best practices for Pods

- Prefer Deployments, StatefulSets, DaemonSets, or Jobs to manage application Pods rather than relying on standalone Pods for long-running production workloads.
- Define CPU and memory requests and appropriate limits.
- Configure readiness, liveness, and startup probes according to application behavior.
- Use non-root containers and a restricted security context where possible.
- Avoid putting passwords or tokens directly in YAML manifests.
- Use ConfigMaps for non-sensitive configuration and Secrets with suitable access controls for sensitive data.
- Set appropriate Pod disruption and availability strategies for the workload.
- Use labels consistently to support selection, monitoring, and operations.
- Collect logs and monitor application health.

## 14. Interview questions

### Kubernetes fundamentals

1. What is Kubernetes?
2. Why do we need container orchestration?
3. What is the difference between Docker and Kubernetes?
4. What is a Kubernetes cluster?
5. What are the main responsibilities of the control plane?

### Kubernetes architecture

6. What does kube-apiserver do?
7. What information is stored in etcd?
8. What is the role of kube-scheduler?
9. What is kubelet?
10. What is reconciliation in Kubernetes?

### Pods

11. What is a Pod?
12. Can a Pod contain multiple containers?
13. How do containers in the same Pod communicate?
14. What is the difference between a Pod and a Deployment?
15. What is the difference between a Pod phase and a displayed status such as `CrashLoopBackOff`?
16. How do you troubleshoot `ImagePullBackOff`?
17. What is the difference between a resource request and a limit?
18. What happens when you delete a standalone Pod versus a Pod managed by a Deployment?

## 15. Combined practical challenge

Complete the following in your local Minikube cluster:

1. Start Minikube and confirm that the node is `Ready`.
2. Inspect the cluster using `kubectl cluster-info` and `kubectl get nodes -o wide`.
3. Inspect system Pods using `kubectl get pods -n kube-system`.
4. Create an NGINX Pod using `kubectl run`.
5. Create a second NGINX Pod using a YAML manifest.
6. Inspect both Pods with `kubectl get pods -o wide` and `kubectl describe pod`.
7. Read the NGINX logs.
8. Use port-forwarding and `curl` to test the NGINX response.
9. Create a Pod with CPU and memory requests and limits.
10. Delete the test Pods and stop Minikube.

### Suggested GitHub screenshots

- `kubectl get nodes -o wide`
- `kubectl get pods -A`
- `kubectl get pods -o wide`
- `kubectl describe pod nginx-pod-yaml`
- NGINX response from `curl`
- Pod resource configuration

## 16. Key takeaways

- Kubernetes orchestrates containerized workloads.
- The control plane manages the cluster; worker nodes run workloads.
- The API server is the main interface to Kubernetes.
- etcd stores cluster state.
- The scheduler selects nodes for Pods.
- kubelet works with the container runtime to run Pods.
- A Pod is the smallest deployable Kubernetes unit.
- Pods can contain one or more closely related containers.
- A standalone Pod is not automatically recreated when deleted.
- Controllers such as Deployments maintain desired replicas and help recover failed Pods.

## 17. Official documentation

- [Kubernetes Overview](https://kubernetes.io/docs/concepts/overview/)
- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Kubernetes Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Debugging Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [Minikube — Get Started](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Coming next — Day 64

**Kubernetes Deployments**

We will learn how Deployments manage replica Pods, perform rolling updates and rollbacks, and help keep applications available.
