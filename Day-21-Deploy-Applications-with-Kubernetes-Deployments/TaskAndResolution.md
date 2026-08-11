# Kubernetes Deployment Task & Resolution Handbook

## 1. Task Objective

Create a Kubernetes Deployment with:

| Requirement     | Value          |
| --------------- | -------------- |
| Deployment name | `nginx`        |
| Application     | Nginx          |
| Image           | `nginx:latest` |
| Replicas        | `1` by default |

The Kubernetes `kubectl` utility is already configured to communicate with the cluster.

---

# 2. Environment

Use the configured Kubernetes jump host.

```text
Administrator
     |
     | kubectl
     v
Kubernetes API Server
     |
     +---- etcd
     |
     +---- Scheduler
     |
     +---- Controllers
     |
     v
Worker Node
     |
     v
Pod
     |
     v
Nginx Container
```

> Server names, usernames, hostnames, and organization-specific names are intentionally generalized.

---

# 3. Create the Deployment

Run:

```bash
kubectl create deployment nginx --image=nginx:latest
```

Expected output:

```text
deployment.apps/nginx created
```

This creates the Deployment in the Kubernetes cluster.

It does **not** create a YAML file in your current Linux directory.

---

# 4. Verify Deployment

```bash
kubectl get deployments
```

Example output:

```text
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           75s
```

### Meaning

* `NAME` → Deployment name
* `READY` → Ready replicas / desired replicas
* `UP-TO-DATE` → Replicas using the current specification
* `AVAILABLE` → Replicas available for use
* `AGE` → Deployment age

`1/1` means the Deployment is healthy.

---

# 5. Verify Pod

```bash
kubectl get pods
```

Example:

```text
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7c5d8bf9f7-w8w6j   1/1     Running   0          97s
```

Important points:

```text
READY     = 1/1
STATUS    = Running
RESTARTS  = 0
```

This confirms that the Nginx container is running successfully.

---

# 6. Verify Image

```bash
kubectl get deployment nginx -o yaml | grep image
```

Expected:

```text
- image: nginx:latest
  imagePullPolicy: Always
```

The important part is:

```text
nginx:latest
```

This confirms that the required image and tag were specified correctly.

---

# 7. Detailed Deployment Verification

```bash
kubectl describe deployment nginx
```

Important information to check:

```text
Name: nginx
Replicas: 1 desired | 1 updated | 1 total | 1 available
```

And:

```text
Image: nginx:latest
```

You can also check the ReplicaSet:

```bash
kubectl get replicasets
```

Example:

```text
NAME               DESIRED   CURRENT   READY
nginx-7c5d8bf9f7   1         1         1
```

---

# 8. Kubernetes Deployment Flow

A Deployment does not directly manage containers.

The normal relationship is:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pod
    |
    v
Container
```

For this task:

```text
Deployment: nginx
       |
       v
ReplicaSet: nginx-7c5d8bf9f7
       |
       v
Pod: nginx-7c5d8bf9f7-w8w6j
       |
       v
Container: nginx
       |
       v
Image: nginx:latest
```

---

# 9. What Happens Internally?

When you run:

```bash
kubectl create deployment nginx --image=nginx:latest
```

the high-level process is:

```text
kubectl
   |
   v
Kubernetes API Server
   |
   v
Deployment stored in etcd
   |
   v
Deployment Controller
   |
   v
ReplicaSet created
   |
   v
Scheduler selects Worker Node
   |
   v
Kubelet receives Pod specification
   |
   v
Container Runtime
   |
   v
nginx:latest image pulled
   |
   v
Nginx container starts
```

---

# 10. Where Is the Deployment Stored?

The Deployment is stored as a Kubernetes object in the cluster.

It is ultimately persisted in:

```text
etcd
```

It is **not stored as a file** in your jump-host directory.

Therefore:

```bash
ls
```

may show no deployment file.

You can retrieve the Kubernetes object with:

```bash
kubectl get deployment nginx -o yaml
```

If you want to save it locally:

```bash
kubectl get deployment nginx -o yaml > nginx-deployment.yaml
```

Now:

```bash
ls
```

will show:

```text
nginx-deployment.yaml
```

---

# 11. Important Commands

## Cluster

```bash
kubectl cluster-info
kubectl get nodes
```

## Deployments

```bash
kubectl get deployments
kubectl describe deployment nginx
kubectl get deployment nginx -o yaml
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## ReplicaSets

```bash
kubectl get replicasets
```

## All resources

```bash
kubectl get all
```

---

# 12. Troubleshooting

## Deployment not found

```bash
kubectl get deployments
```

Check all namespaces:

```bash
kubectl get deployments -A
```

---

## Pod is Pending

```bash
kubectl describe pod <pod-name>
```

Check:

* Insufficient CPU
* Insufficient memory
* Taints
* Node availability
* Scheduling constraints

---

## Pod is CrashLoopBackOff

```bash
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

Typical causes:

* Application failure
* Incorrect command
* Configuration error
* Missing environment variable
* Permission problem

---

## ImagePullBackOff

Check:

```bash
kubectl describe pod <pod-name>
```

Typical causes:

* Incorrect image name
* Incorrect tag
* Private registry authentication
* Registry unavailable

For this task verify:

```text
nginx:latest
```

---

## Deployment Available but Pod Not Running

Run:

```bash
kubectl get deployment
kubectl get replicasets
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 13. Common Mistakes

### Incorrect command

```bash
kubectl create deployment nginx --image=nginx
```

### Required command

```bash
kubectl create deployment nginx --image=nginx:latest
```

---

### Typographical error

Incorrect:

```bash
keubectl
```

Correct:

```bash
kubectl
```

---

# 14. Production Best Practice

Although this task requires:

```text
nginx:latest
```

using `latest` is generally **not recommended in production**.

Prefer an immutable version:

```text
nginx:1.27.2
```

Why?

If `latest` changes, the same Kubernetes YAML can potentially deploy different application versions at different times.

Versioned images provide:

* Reproducibility
* Easier rollback
* Better auditing
* Predictable deployments

---

# 15. Deployment vs Pod

| Feature                           | Pod     | Deployment  |
| --------------------------------- | ------- | ----------- |
| Runs containers                   | Yes     | Indirectly  |
| Self-healing                      | Limited | Yes         |
| Replica management                | No      | Yes         |
| Rolling updates                   | No      | Yes         |
| Rollback                          | No      | Yes         |
| Production application management | Limited | Recommended |

---

# 16. Deployment vs ReplicaSet

```text
Deployment
    |
    | manages
    v
ReplicaSet
    |
    | manages
    v
Pods
```

A Deployment provides higher-level functionality such as:

* Rolling updates
* Rollbacks
* Replica management
* Version management

---

# 17. Kubernetes Interview Questions

## L1 — Junior

### Q1. What is Kubernetes?

Kubernetes is a container orchestration platform used to deploy, manage, scale, and maintain containerized applications.

---

### Q2. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

It contains one or more containers that share networking and storage resources.

---

### Q3. What is a Deployment?

A Deployment manages the desired state of application Pods and provides:

* Replica management
* Rolling updates
* Rollbacks
* Self-healing through ReplicaSets

---

### Q4. What is a ReplicaSet?

A ReplicaSet ensures that the specified number of Pods are running.

Example:

```text
Desired replicas = 3
Running replicas = 2
```

ReplicaSet creates another Pod.

```text
2 → 3
```

---

# 18. L2 — Intermediate

### Q5. Why does a Deployment create a ReplicaSet?

The Deployment uses ReplicaSets to manage Pods.

This allows Kubernetes to maintain the desired replica count while also supporting rolling updates and rollbacks.

---

### Q6. What happens when a Pod is deleted?

If the Pod belongs to a Deployment:

```text
Pod deleted
   |
   v
ReplicaSet detects fewer replicas
   |
   v
New Pod created
```

This is Kubernetes self-healing.

---

### Q7. How do you scale a Deployment?

```bash
kubectl scale deployment nginx --replicas=3
```

Verify:

```bash
kubectl get deployment nginx
kubectl get pods
```

---

### Q8. How do you update the image?

```bash
kubectl set image deployment/nginx nginx=nginx:1.27.2
```

Check:

```bash
kubectl rollout status deployment/nginx
```

---

### Q9. How do you rollback?

```bash
kubectl rollout undo deployment/nginx
```

Check:

```bash
kubectl rollout status deployment/nginx
```

---

# 19. L3 — Senior

### Q10. A Deployment shows 3 desired replicas but only 2 are available. What do you check?

Start with:

```bash
kubectl get deployment
kubectl get replicasets
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Investigate:

* Scheduling
* Resource limits
* Image pull
* Readiness probes
* Liveness probes
* Application startup
* Node health

---

### Q11. Pods are Running but users cannot access the application. What do you investigate?

Check:

```text
Pod
 ↓
Service
 ↓
Endpoints
 ↓
Ingress / Load Balancer
 ↓
Network
```

Commands:

```bash
kubectl get svc
kubectl get endpoints
kubectl describe svc <service>
kubectl get ingress
```

Then investigate:

* Security groups
* Network ACLs
* Load balancer
* DNS
* Application listening port

---

### Q12. How does Kubernetes perform self-healing?

Kubernetes controllers continuously compare:

```text
Desired State
      vs
Actual State
```

If they differ, controllers take corrective action.

Example:

```text
Desired = 3 Pods
Actual  = 2 Pods

Controller creates 1 Pod

Actual = 3 Pods
```

---

# 20. L4 — Architect

### Q13. How would you design a production Kubernetes application?

A typical architecture:

```text
Internet
   |
Route 53
   |
Load Balancer
   |
Ingress
   |
Kubernetes Service
   |
Deployment
   |
Pods
   |
Application
```

Supporting components:

```text
CI/CD
  |
Container Registry
  |
Kubernetes

Monitoring
  |
Metrics + Logs + Alerts

Security
  |
IAM + RBAC + Network Policies + Secrets
```

---

### Q14. What are the key production considerations?

Important areas:

* High availability
* Multi-AZ worker nodes
* Autoscaling
* Resource requests/limits
* Readiness probes
* Liveness probes
* Security
* Network policies
* Secrets management
* Monitoring
* Logging
* Backup
* Disaster recovery
* CI/CD
* Image vulnerability scanning

---

# 21. AWS EKS Mapping

Amazon EKS is AWS's managed Kubernetes service.

Conceptual mapping:

| Kubernetes               | AWS                              |
| ------------------------ | -------------------------------- |
| Kubernetes Control Plane | Amazon EKS managed control plane |
| Worker Node              | EC2 / EKS managed node group     |
| Container Image          | Amazon ECR                       |
| LoadBalancer Service     | AWS Load Balancer                |
| IAM                      | IAM / EKS access                 |
| Networking               | Amazon VPC                       |
| Monitoring               | CloudWatch                       |
| DNS                      | Route 53                         |
| Storage                  | EBS / EFS                        |
| Autoscaling              | EKS/EC2 autoscaling components   |

---

# 22. AWS Scenario

## Scenario

An Nginx Deployment runs on Amazon EKS.

```text
Internet
    |
Route 53
    |
AWS Load Balancer
    |
Kubernetes Service
    |
Deployment
    |
ReplicaSet
    |
Pods
    |
Nginx Containers
```

The worker nodes run inside an AWS VPC.

---

# 23. AWS Interview Question

### Q15. What happens if an EKS Pod crashes?

The Kubernetes control plane detects that the desired state is not satisfied.

If the Pod is managed by a Deployment:

```text
Pod crashes
   |
   v
ReplicaSet detects missing Pod
   |
   v
Replacement Pod created
   |
   v
Scheduler selects suitable node
   |
   v
Kubelet starts container
```

---

# 24. AWS Incident RCA Example

## Incident

Application Pods are running but users receive HTTP 503.

### Symptoms

```text
Pods: Running
Deployment: Healthy
Service: Exists
Users: HTTP 503
```

### Investigation

Check:

```bash
kubectl get pods
kubectl get svc
kubectl get endpoints
```

Suppose:

```text
Service: nginx
Endpoints: <none>
```

### Root Cause

The Service selector does not match the Pod labels.

Example:

Service:

```yaml
selector:
  app: web
```

Pod:

```yaml
labels:
  app: nginx
```

No matching Pods are selected.

### Resolution

Make the selectors consistent.

```yaml
selector:
  app: nginx
```

### Lesson

A healthy Pod does not guarantee that application traffic can reach the Pod.

Always check:

```text
Pod
Service
Endpoints
Ingress / Load Balancer
Network
```

---

# 25. DevOps Mock Interview

## Interviewer

Create an Nginx Deployment.

### Candidate

```bash
kubectl create deployment nginx --image=nginx:latest
```

Then:

```bash
kubectl get deployment nginx
kubectl get pods
```

I would verify that the Deployment is available and the Pod is running.

---

## Interviewer

The Pod is `Pending`. What do you do?

### Candidate

I would first inspect the Pod:

```bash
kubectl describe pod <pod-name>
```

Then check:

```bash
kubectl get nodes
kubectl get events --sort-by=.metadata.creationTimestamp
```

I would look for:

* Insufficient CPU/memory
* Taints
* Affinity rules
* Node availability
* Scheduling failures

---

## Interviewer

The Pod is `CrashLoopBackOff`.

### Candidate

I would check:

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

`--previous` is especially useful because it retrieves logs from the previous crashed container instance.

I would then investigate the application error, configuration, command, environment variables, probes, and permissions.

---

## Interviewer

The Pod is Running but application is unavailable.

### Candidate

I would not assume the application is healthy merely because the Pod is Running.

I would check:

```bash
kubectl get svc
kubectl get endpoints
kubectl describe svc <service-name>
kubectl get ingress
```

Then verify:

* Service selector
* Target port
* Container port
* Readiness probe
* Load balancer
* DNS
* Security groups
* Network policies

---

# 26. Most Important Kubernetes Commands Cheat Sheet

```bash
# Cluster
kubectl cluster-info
kubectl get nodes

# Deployments
kubectl get deployments
kubectl describe deployment nginx
kubectl get deployment nginx -o yaml

# Create
kubectl create deployment nginx --image=nginx:latest

# Pods
kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous

# ReplicaSets
kubectl get replicasets

# Scaling
kubectl scale deployment nginx --replicas=3

# Update
kubectl set image deployment/nginx nginx=nginx:1.27.2

# Rollout
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx

# Services
kubectl get svc
kubectl describe svc <service>

# Events
kubectl get events --sort-by=.metadata.creationTimestamp

# All resources
kubectl get all
```

---

# 27. Final Resolution for This Task

The required command is:

```bash
kubectl create deployment nginx --image=nginx:latest
```

Validation:

```bash
kubectl get deployment nginx
kubectl get pods
kubectl get deployment nginx -o yaml | grep image
```

Expected image:

```text
nginx:latest
```

Expected Deployment state:

```text
1/1 READY
```

Expected Pod state:

```text
Running
```

Therefore, the task is successfully completed when these conditions are satisfied.

---

# 28. Quick Interview Revision

Remember this flow:

```text
kubectl
   ↓
API Server
   ↓
etcd
   ↓
Deployment Controller
   ↓
ReplicaSet
   ↓
Scheduler
   ↓
Worker Node
   ↓
Kubelet
   ↓
Container Runtime
   ↓
Pod
   ↓
Container
```

Remember these relationships:

```text
Deployment → ReplicaSet → Pod → Container
```

Remember these troubleshooting commands:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
```

Remember production image best practice:

```text
Avoid:
nginx:latest

Prefer:
nginx:<specific-version>
```

Remember the AWS equivalent:

```text
Kubernetes
    ↓
Amazon EKS

Worker Nodes
    ↓
EC2 / Managed Node Groups

Container Registry
    ↓
Amazon ECR

Networking
    ↓
Amazon VPC

Monitoring
    ↓
CloudWatch
```

---

# 29. Final Key Takeaways

1. A Deployment manages application Pods.
2. A Deployment creates/manages a ReplicaSet.
3. A ReplicaSet maintains the desired number of Pods.
4. Pods contain containers.
5. `kubectl` communicates with the Kubernetes API Server.
6. Cluster state is persisted in `etcd`.
7. The Scheduler selects appropriate worker nodes.
8. Kubelet manages Pods on worker nodes.
9. Kubernetes continuously reconciles desired and actual state.
10. Deployments provide rolling updates and rollbacks.
11. `Running` does not necessarily mean the application is reachable.
12. For production, use immutable image versions instead of `latest`.
13. In AWS, Amazon EKS provides managed Kubernetes control-plane capabilities.
14. Production troubleshooting should follow the path:

```text
Pod → Service → Endpoint → Ingress/Load Balancer → Network → DNS
```

15. The most important DevOps skill is not memorizing commands; it is understanding **why Kubernetes behaves the way it does and systematically troubleshooting from symptoms to root cause**.

