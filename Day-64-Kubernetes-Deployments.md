# Day 64/100 — Kubernetes Deployments

**GitHub filename:** `day64-kubernetes-deployments.md`

## Learning objectives
- Explain Deployments, ReplicaSets, and Pods.
- Create a Deployment using kubectl and YAML.
- Scale applications.
- Perform rolling updates and rollbacks.
- Troubleshoot common Deployment issues.

## 1. What is a Kubernetes Deployment?

A Deployment is a Kubernetes workload resource that manages a set of identical Pods and provides declarative application updates. You specify the desired state, such as three replicas, and Kubernetes controllers work to maintain it.

If a Pod managed by a Deployment disappears, the controller chain works to create a replacement.

**Real-world example:** An e-commerce application runs three web replicas. If one Pod fails, Kubernetes works to restore the desired replica count.

## 2. Why use Deployments?

| Without Deployment | With Deployment |
|---|---|
| Manually manage Pods | Manage Pods declaratively |
| Manually replace deleted Pods | Controllers maintain replica count |
| Manual updates | Rolling updates |
| Manual rollback process | Rollout history and undo |
| Difficult replica management | Easy scaling |

## 3. Deployment architecture

```text
             Deployment
                 |
                 v
             ReplicaSet
                 |
          +------+------+------+
          |      |      |      |
          v      v      v
        Pod 1  Pod 2  Pod 3
```

- **Deployment:** Manages desired application state and rollout history.
- **ReplicaSet:** Maintains the requested number of matching Pods.
- **Pod:** Runs one or more application containers.

When the Pod template changes, Kubernetes generally creates a new ReplicaSet and gradually replaces old Pods according to the rollout strategy.

## 4. Create a Deployment using kubectl

Start Minikube:

```bash
minikube start --driver=docker
```

Create an NGINX Deployment with three replicas:

```bash
kubectl create deployment nginx-deployment --image=nginx:1.27 --replicas=3
```

Inspect it:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods -o wide
```

Expected: three Pods should eventually show `Running` and `1/1` Ready. Pod names and IP addresses vary.

## 5. Create a Deployment using YAML

Create `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

Apply and inspect:

```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployment nginx-deployment
kubectl get replicasets
kubectl get pods --show-labels
```

Important fields:

| Field | Purpose |
|---|---|
| `apiVersion` | Deployment API version |
| `kind` | Resource type |
| `metadata.name` | Deployment name |
| `spec.replicas` | Desired number of Pods |
| `selector.matchLabels` | Selects the Pods managed by the Deployment |
| `template` | Pod template used to create replicas |
| `containers.image` | Container image |
| `resources.requests` | Resource amount used for scheduling |
| `resources.limits` | Container resource ceiling |

The selector must match the labels in the Pod template. A Deployment's selector is generally immutable after creation, so choose it carefully.

## 6. Scale a Deployment

Scale from three to five replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get deployment nginx-deployment
kubectl get pods
```

Scale down to two:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Kubernetes controllers work asynchronously to reach the requested replica count.

## 7. Update the application image

Update NGINX from 1.27 to 1.28:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.28
```

Monitor the rollout:

```bash
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl describe deployment nginx-deployment
```

A rolling update gradually creates new Pods and scales down old Pods, subject to the Deployment strategy, readiness, and available resources.

## 8. Rolling update strategy

Two important settings are:

- `maxSurge`: How many Pods above the desired replica count may be created during an update.
- `maxUnavailable`: How many Pods may be unavailable during an update.

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

With three desired replicas, this allows one extra Pod during the rollout and aims to keep all three desired replicas available. Progress still depends on readiness and cluster capacity.

## 9. Roll back a Deployment

If a new version is faulty, inspect rollout history:

```bash
kubectl rollout history deployment/nginx-deployment
```

Undo the latest rollout:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Verify:

```bash
kubectl rollout status deployment/nginx-deployment
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment
```

To roll back to a particular revision, inspect the history first, then use the appropriate revision number:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

Revision numbers depend on the Deployment's history; do not assume revision 1 is always the desired version.

## 10. Pause and resume a rollout

```bash
kubectl rollout pause deployment/nginx-deployment
kubectl set image deployment/nginx-deployment nginx=nginx:1.28
kubectl rollout resume deployment/nginx-deployment
kubectl rollout status deployment/nginx-deployment
```

Use this carefully and understand which changes are grouped into the rollout.

## 11. Expose the Deployment using a Service

A Deployment manages Pods but does not itself provide a stable network endpoint. A Service provides stable access to matching Pods.

Create a ClusterIP Service:

```bash
kubectl expose deployment nginx-deployment --name=nginx-service --port=80 --target-port=80 --type=ClusterIP
```

Inspect it:

```bash
kubectl get services
kubectl describe service nginx-service
```

Test locally using port-forwarding:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Keep that terminal running. In another terminal:

```bash
curl http://127.0.0.1:8080
```

You should receive the NGINX welcome page HTML. Stop port-forwarding with `Ctrl+C`.

## 12. Useful Deployment commands

| Command | Purpose |
|---|---|
| `kubectl get deployments` | List Deployments |
| `kubectl describe deployment <name>` | Inspect details and events |
| `kubectl get replicasets` | List ReplicaSets |
| `kubectl get pods -o wide` | List Pods and node placement |
| `kubectl scale deployment <name> --replicas=5` | Change desired replicas |
| `kubectl set image deployment/<name> <container>=<image>` | Update image |
| `kubectl rollout status deployment/<name>` | Monitor rollout |
| `kubectl rollout history deployment/<name>` | View rollout history |
| `kubectl rollout undo deployment/<name>` | Undo latest rollout |
| `kubectl delete deployment <name>` | Delete Deployment |

## 13. Troubleshooting

| Symptom | Possible cause | What to investigate |
|---|---|---|
| Fewer available replicas | Pods Pending, failing, or not ready | `kubectl get pods`; `kubectl describe deployment` |
| `ImagePullBackOff` | Invalid image/tag or registry access | `kubectl describe pod <pod-name>` |
| `CrashLoopBackOff` | Application repeatedly exits | `kubectl logs <pod-name>` and `kubectl logs <pod-name> --previous` |
| Rollout stuck | New Pods not Ready or insufficient resources | `kubectl rollout status` and Pod events |
| Service selects no Pods | Selector and labels do not match | `kubectl get pods --show-labels`; `kubectl describe service` |
| Pods remain Pending | Insufficient resources or scheduling constraints | `kubectl describe pod <pod-name>` |

Useful commands:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods -o wide
kubectl describe deployment nginx-deployment
kubectl rollout status deployment/nginx-deployment
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 14. Production best practices

- Use declarative manifests and store them in version control.
- Set appropriate CPU and memory requests and limits.
- Configure readiness, liveness, and startup probes based on application behavior.
- Choose rolling update settings that suit availability requirements.
- Ensure the cluster has capacity for rollout surge.
- Use trusted image registries and controlled image versions or immutable digests.
- Apply least-privilege RBAC and secure secrets.
- Monitor rollout status, Pod health, and application metrics.
- Test rollback procedures.
- Consider PodDisruptionBudgets and topology spread constraints for availability.

## 15. Interview questions

1. What is a Kubernetes Deployment?
2. What is the difference between a Pod, ReplicaSet, and Deployment?
3. How does a Deployment replace a failed Pod?
4. How do you scale a Deployment?
5. What is a rolling update?
6. What are `maxSurge` and `maxUnavailable`?
7. How do you check rollout status?
8. How do you roll back a Deployment?
9. What happens when you change the image in a Deployment?
10. Why use a Deployment instead of standalone Pods for long-running applications?
11. What is the relationship between a Deployment and a Service?
12. How would you investigate a Deployment stuck during rollout?

## 16. Practical challenge

Using Minikube:

- [ ] Start Minikube and confirm the node is `Ready`.
- [ ] Create an NGINX Deployment with three replicas using YAML.
- [ ] Verify the Deployment, ReplicaSet, and Pods.
- [ ] Scale the Deployment to five replicas.
- [ ] Update the image from `nginx:1.27` to `nginx:1.28`.
- [ ] Monitor the rollout and inspect its history.
- [ ] Roll back to the previous revision.
- [ ] Expose the Deployment with a ClusterIP Service.
- [ ] Test it using port-forwarding and `curl`.
- [ ] Delete the Service and Deployment when finished.

### Suggested GitHub screenshots

- `kubectl get deployments`
- `kubectl get replicasets`
- `kubectl get pods -o wide`
- Deployment after scaling to five replicas
- `kubectl rollout status`
- `kubectl rollout history`
- Successful rollback
- NGINX response through Service port-forwarding

## 17. Key takeaways

- A Deployment manages replicated Pods declaratively.
- A Deployment creates and manages ReplicaSets.
- ReplicaSets maintain the desired number of Pods.
- Scaling changes the desired replica count.
- Rolling updates release new versions gradually.
- Rollout history and undo support rollback workflows.
- Services provide stable network access to Pods; Deployments do not replace Services.

## 18. Official documentation

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)
- [Rolling Back a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Minikube — Get Started](https://minikube.sigs.k8s.io/docs/start/)

## Coming next — Day 65

**Kubernetes Services**

We will learn ClusterIP, NodePort, and LoadBalancer Services, selectors, ports, and how to expose applications inside and outside a cluster.
