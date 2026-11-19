# Day 61/100 — Introduction to Kubernetes

**Series:** 100 Days of DevOps — From Zero to Hero  
**Topic:** Introduction to Kubernetes  
**Suggested filename:** `day61-introduction-to-kubernetes.md`

## Learning objectives

By the end of this lesson, you should be able to:

- Explain what Kubernetes is and why container orchestration is useful.
- Describe common Kubernetes capabilities.
- Understand basic Kubernetes terminology.
- Distinguish container tooling from Kubernetes orchestration.
- Start a local Kubernetes cluster and deploy a simple application.

## 1. What is Kubernetes?

Kubernetes (often shortened to **K8s**) is an open-source platform for orchestrating containerized applications. It helps automate deployment, scaling, networking, and recovery of workloads.

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

The control plane manages the cluster. Worker nodes run the application workloads.

We will explore these components in detail in **Day 62 — Kubernetes Architecture**.

## 6. Hands-on lab: run your first Kubernetes application

This lab uses Minikube with Docker Desktop. It is suitable for a local learning environment on macOS (including Apple Silicon) or Windows.

### Prerequisites

Install and start:

- Docker Desktop
- `kubectl`
- Minikube

Check that the tools are available:

```bash
docker version
kubectl version --client
minikube version
```

If Docker Desktop is your Minikube driver, make sure its engine is running before you start the cluster.

### Step 1 — Start a local cluster

```bash
minikube start --driver=docker
```

Check its status:

```bash
minikube status
kubectl cluster-info
```

### Step 2 — View the cluster nodes

```bash
kubectl get nodes
```

You should normally see a local node named `minikube` with status `Ready` once startup has completed.

### Step 3 — Create a Deployment

Create an NGINX Deployment with two replicas:

```bash
kubectl create deployment nginx --image=nginx:1.27 --replicas=2
```

A Deployment manages the Pods for this application.

### Step 4 — Check the Deployment and Pods

```bash
kubectl get deployments
kubectl get pods -o wide
```

Wait until both Pods show `Running` and their readiness column shows `1/1`.

You can watch the Pods:

```bash
kubectl get pods -w
```

Press `Ctrl+C` to stop watching.

### Step 5 — Describe the Deployment

```bash
kubectl describe deployment nginx
```

Review the desired and available replica counts, selector, Pod template, and events.

### Step 6 — Expose the Deployment inside the cluster

```bash
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

Check the Service:

```bash
kubectl get services
kubectl describe service nginx
```

A ClusterIP Service is reachable from within the cluster. To test it locally, start a temporary port-forward:

```bash
kubectl port-forward service/nginx 8080:80
```

Keep that terminal running. In a second terminal, run:

```bash
curl http://127.0.0.1:8080
```

You should receive the NGINX welcome page HTML. Stop port-forwarding with `Ctrl+C`.

### Step 7 — Scale the Deployment

Increase the number of replicas to three:

```bash
kubectl scale deployment nginx --replicas=3
kubectl get pods
```

Kubernetes should work towards three desired replicas.

### Step 8 — Inspect logs

Choose one Pod name from `kubectl get pods`, then run:

```bash
kubectl logs <pod-name>
```

This displays the container's standard output and error logs.

### Step 9 — Clean up

Delete the Service and Deployment:

```bash
kubectl delete service nginx
kubectl delete deployment nginx
```

Stop Minikube when finished:

```bash
minikube stop
```

The commands above create a learning workload only. Do not expose a test application publicly or use this local setup as a production cluster.

## 7. Common beginner troubleshooting

| Symptom | What to check |
|---|---|
| Minikube does not start | Confirm Docker Desktop is running and check `minikube status` |
| Node is `NotReady` | Run `kubectl describe node <node-name>` |
| Pod is `Pending` | Inspect scheduling events with `kubectl describe pod <pod-name>` |
| Pod is `ImagePullBackOff` | Check the image name, tag, registry access, and pull credentials |
| Pod is `CrashLoopBackOff` | Inspect `kubectl logs <pod-name>` and `kubectl describe pod <pod-name>` |
| Service cannot be reached | Check Service selectors, Pod labels, endpoints, and port-forward command |
| kubectl uses the wrong cluster | Check `kubectl config current-context` |

Useful commands:

```bash
kubectl get all
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl config current-context
```

## 8. Production best practices

- Use Deployments or other suitable workload controllers for long-running applications rather than relying on standalone Pods.
- Set appropriate CPU and memory requests and limits.
- Configure readiness, liveness, and startup probes according to application behavior.
- Use non-root containers and restricted security contexts where possible.
- Avoid putting passwords or tokens directly in YAML manifests.
- Use ConfigMaps for non-sensitive configuration and Secrets with suitable access controls for sensitive data.
- Monitor application and cluster health.
- Plan upgrades, capacity, and recovery procedures.

## 9. Interview questions

1. What is Kubernetes?
2. Why do we need container orchestration?
3. What is a Kubernetes cluster?
4. What is the difference between a Pod and a container?
5. What is a Deployment?
6. What is a Kubernetes Service?
7. What is self-healing in Kubernetes?
8. How can you scale an application in Kubernetes?
9. Does Kubernetes require Docker Engine as its container runtime?
10. What is the difference between Docker Compose and Kubernetes?

## 10. Practical challenge

Complete the following in your local cluster:

1. Start Minikube and confirm the node is `Ready`.
2. Inspect the cluster using `kubectl cluster-info` and `kubectl get nodes`.
3. Create an NGINX Deployment with two replicas.
4. Verify that both Pods are ready.
5. Expose the Deployment using a ClusterIP Service.
6. Use port-forwarding and `curl` to test the application.
7. Scale the Deployment to three replicas.
8. Inspect one Pod's logs and Deployment details.
9. Delete the Service and Deployment, then stop Minikube.

### Suggested GitHub screenshots

- `kubectl get nodes`
- `kubectl get deployments`
- `kubectl get pods -o wide`
- `kubectl get services`
- NGINX response from the port-forward test
- Deployment after scaling to three replicas

## 11. Key takeaways

- Kubernetes orchestrates containerized applications.
- It helps with scheduling, scaling, service discovery, rolling updates, and recovery.
- A Pod is the smallest deployable Kubernetes unit.
- Deployments manage replicated Pods and application updates.
- Services provide stable network access to workloads.
- Kubernetes uses declarative desired state and controllers to reconcile the cluster.

## 12. Official documentation

- [Kubernetes Overview](https://kubernetes.io/docs/concepts/overview/)
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
- [Minikube — Get Started](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Coming next

**Day 62 — Kubernetes Architecture**

We will explore the control plane, worker node components, API request flow, etcd, scheduler, kubelet, and how Kubernetes keeps actual state aligned with desired state.
