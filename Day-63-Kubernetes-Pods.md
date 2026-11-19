# Day 63/100 — Kubernetes Pods

**Series:** 100 Days of DevOps — From Zero to Hero  
**Topic:** Kubernetes Pods  
**Suggested filename:** `day63-kubernetes-pods.md`

## Learning objectives

By the end of this lesson, you should be able to:

- Explain what a Pod is and why Kubernetes uses Pods.
- Distinguish a Pod, container, and node.
- Create Pods using `kubectl` and YAML manifests.
- Inspect Pod status, events, and logs.
- Execute commands inside a Pod.
- Access a Pod locally using port-forwarding.
- Understand common Pod states and troubleshoot failures.
- Configure basic resource requests and limits.

## 1. What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes. Kubernetes schedules Pods onto worker nodes, and each Pod contains one or more containers that are tightly coupled.

Most application Pods contain one main application container. Some Pods contain additional helper containers, often called sidecars.

Containers in the same Pod:

- Share the same network namespace and Pod IP address.
- Can communicate with each other through `localhost`.
- Can share storage volumes when configured.
- Are scheduled together on the same node.
- Have separate container filesystems unless storage is shared or mounted.

A Pod is not a permanent machine. It can be replaced, and a replacement Pod can have a different IP address.

## 2. Pod vs container vs node

| Item | Meaning | Example |
|---|---|---|
| Node | Machine in the cluster | Worker virtual machine |
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

## 6. Common Pod phases and displayed statuses

The Pod **phase** is a high-level summary. `kubectl get pods` also displays a STATUS column that can show more specific information about container startup and failure.

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

View logs:

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

Not every container image includes `/bin/sh` or debugging utilities. Minimal or distroless images may require a different debugging approach.

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

Create `nginx-resources.yaml`:

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
- Use labels consistently to support selection, monitoring, and operations.
- Collect logs and monitor application health.
- Plan Pod availability and disruption handling according to application requirements.

## 14. Interview questions

1. What is a Pod?
2. Can a Pod contain multiple containers?
3. How do containers in the same Pod communicate?
4. What is the difference between a Pod, container, and node?
5. What is the difference between a Pod and a Deployment?
6. What is the difference between a Pod phase and a displayed status such as `CrashLoopBackOff`?
7. How do you troubleshoot `ImagePullBackOff`?
8. How do you view logs from a previous container instance?
9. What is the difference between a resource request and a limit?
10. What happens when you delete a standalone Pod versus a Pod managed by a Deployment?
11. What does `kubectl describe pod` show?
12. What is the purpose of port-forwarding?

## 15. Practical challenge

Complete the following in your local Minikube cluster:

1. Start Minikube and confirm the node is `Ready`.
2. Create an NGINX Pod using `kubectl run`.
3. Create a second NGINX Pod using the YAML manifest.
4. Inspect both Pods with `kubectl get pods -o wide` and `kubectl describe pod`.
5. Read the NGINX logs.
6. Use port-forwarding and `curl` to test the NGINX response.
7. Create a Pod with CPU and memory requests and limits.
8. Inspect the resource configuration.
9. Delete the test Pods and stop Minikube.

### Suggested GitHub screenshots

- `kubectl get pods -o wide`
- `kubectl describe pod nginx-pod-yaml`
- `kubectl get pods --show-labels`
- NGINX response from `curl`
- Pod resource configuration

## 16. Key takeaways

- A Pod is the smallest deployable Kubernetes unit.
- A Pod can contain one or more closely related containers.
- Containers in the same Pod share networking and can share configured storage.
- A standalone Pod is not automatically recreated when deleted.
- Controllers such as Deployments maintain desired replicas and help recover failed Pods.
- `kubectl get`, `describe`, `logs`, and `exec` are essential Pod troubleshooting commands.
- Resource requests and limits help manage cluster capacity and container resource usage.

## 17. Official documentation

- [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Debugging Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [Container Resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Minikube — Get Started](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Coming next — Day 64

**Kubernetes Deployments**

We will learn how Deployments manage replica Pods, perform rolling updates and rollbacks, and help keep applications available.
