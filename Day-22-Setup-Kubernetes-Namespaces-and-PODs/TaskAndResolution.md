# Kubernetes Namespace & Pod — Task and Resolution Handbook

## 1. Task Overview

### Objective

Create a Kubernetes Namespace named:

```text
dev
```

Then create a Pod inside that namespace with:

| Requirement     | Value               |
| --------------- | ------------------- |
| Namespace       | `dev`               |
| Pod name        | `dev-nginx-pod`     |
| Image           | `nginx:latest`      |
| Image tag       | Explicitly `latest` |
| Expected status | `Running`           |

The Kubernetes cluster is already configured and `kubectl` is available.

---

# 2. Kubernetes Architecture Relevant to This Task

```text
Kubernetes Cluster
│
├── Namespace: default
│
├── Namespace: dev
│   │
│   └── Pod: dev-nginx-pod
│       │
│       └── Container
│           │
│           └── nginx:latest
│
├── Namespace: kube-system
│
├── Namespace: kube-public
│
└── Namespace: kube-node-lease
```

The important relationship is:

```text
Cluster
   |
   +-- Namespace
          |
          +-- Pod
                 |
                 +-- Container
                        |
                        +-- Image
```

---

# 3. Namespace

A Namespace is a logical partition inside a Kubernetes cluster.

It is commonly used to separate:

```text
dev
test
staging
prod
```

Example:

```text
Kubernetes Cluster
│
├── dev
│   ├── frontend
│   └── backend
│
├── test
│   ├── frontend
│   └── backend
│
└── prod
    ├── frontend
    └── backend
```

Namespaces are useful for:

* Logical isolation
* RBAC
* Resource quotas
* Environment separation
* Application organization

A Namespace is not the same as a separate Kubernetes cluster. Stronger isolation can require RBAC, NetworkPolicies, dedicated nodes, cloud-account separation, or separate clusters depending on the security requirement.

---

# 4. Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain:

```text
Pod
├── Container 1
├── Container 2
└── Shared resources
```

Most simple applications use one container per Pod:

```text
Pod
└── nginx container
```

Containers inside the same Pod share the Pod's network namespace and can share mounted volumes.

---

# 5. Step 1 — Check Existing Namespaces

Command:

```bash
kubectl get namespaces
```

Example output:

```text
NAME              STATUS   AGE
default           Active   31m
kube-node-lease   Active   31m
kube-public       Active   31m
kube-system       Active   31m
```

### Explanation

The cluster initially contains standard namespaces.

The required `dev` namespace does not yet exist.

---

# 6. Step 2 — Create the Namespace

Command:

```bash
kubectl create namespace dev
```

Output:

```text
namespace/dev created
```

This creates a Kubernetes Namespace object named `dev`.

Verify:

```bash
kubectl get namespaces
```

Example:

```text
NAME              STATUS   AGE
default           Active   32m
dev               Active   9s
kube-node-lease   Active   32m
kube-public       Active   32m
kube-system       Active   32m
```

The important result is:

```text
dev    Active
```

---

# 7. Step 3 — Create the Pod

Command:

```bash
kubectl run dev-nginx-pod \
  --image=nginx:latest \
  --namespace=dev
```

Output:

```text
pod/dev-nginx-pod created
```

### What this command specifies

```text
Pod name      = dev-nginx-pod
Image         = nginx:latest
Namespace     = dev
```

The `--image` option is particularly important because the task explicitly requires:

```text
nginx:latest
```

and not simply:

```text
nginx
```

---

# 8. Step 4 — Verify the Pod

Command:

```bash
kubectl get pods -n dev
```

Output:

```text
NAME             READY   STATUS    RESTARTS   AGE
dev-nginx-pod    1/1     Running   0          78s
```

### Interpretation

```text
READY     = 1/1
STATUS    = Running
RESTARTS  = 0
```

This indicates that the container is running and ready.

---

# 9. Step 5 — Verify the Image

Command:

```bash
kubectl get pod dev-nginx-pod -n dev -o yaml | grep image
```

Output:

```text
- image: nginx:latest
  imagePullPolicy: Always
  image: docker.io/library/nginx:latest
  imageID: docker.io/library/nginx@sha256:...
```

The important line is:

```text
image: nginx:latest
```

This confirms that the required image tag was explicitly specified.

---

# 10. Why Are There Multiple Image Lines?

You may see:

```text
nginx:latest
```

and:

```text
docker.io/library/nginx:latest
```

and:

```text
docker.io/library/nginx@sha256:...
```

These represent different stages of image identification.

### Requested image

```text
nginx:latest
```

This is what was specified in the Pod specification.

### Fully qualified image

```text
docker.io/library/nginx:latest
```

This identifies the registry/repository used to obtain the image.

### Immutable image digest

```text
docker.io/library/nginx@sha256:<digest>
```

The digest identifies the exact image content that was pulled.

Therefore:

```text
Tag
 ↓
nginx:latest
 ↓
Resolved image
 ↓
SHA256 digest
```

---

# 11. Step 6 — Verify Pod Networking

Command:

```bash
kubectl get pod dev-nginx-pod -n dev -o wide
```

Example:

```text
NAME             READY   STATUS    RESTARTS   AGE     IP         NODE
dev-nginx-pod    1/1     Running   0          4m11s   10.22.0.9  worker-node
```

Important information:

```text
Pod IP = 10.22.0.9
Node   = worker-node
```

The actual IP and node name can differ between clusters.

---

# 12. Step 7 — Detailed Pod Inspection

Command:

```bash
kubectl describe pod dev-nginx-pod -n dev
```

Important sections include:

```text
Name:             dev-nginx-pod
Namespace:        dev
Status:           Running
IP:               10.22.0.9
```

Container:

```text
Containers:
  dev-nginx-pod:
    Image: nginx:latest
    State: Running
    Ready: True
    Restart Count: 0
```

---

# 13. Understanding the Container

The command:

```bash
kubectl run dev-nginx-pod --image=nginx:latest --namespace=dev
```

creates the Pod specification.

There is no separate command in the task to create the container.

The high-level flow is:

```text
kubectl
   |
   v
Kubernetes API Server
   |
   v
Pod object
   |
   v
Scheduler
   |
   v
Worker Node
   |
   v
Kubelet
   |
   v
Container Runtime
   |
   v
Container
```

In this cluster, the container runtime shown in the output was:

```text
containerd
```

The `describe` output contained:

```text
Container ID:
containerd://...
```

This means the kubelet used containerd to create and run the actual container.

---

# 14. Pod Creation vs Container Creation

This is an important interview concept.

### User command

```bash
kubectl run dev-nginx-pod --image=nginx:latest -n dev
```

Creates the Kubernetes Pod specification.

### Kubernetes components

The Kubernetes control plane schedules the Pod.

### Kubelet

The kubelet on the selected worker node receives the Pod specification.

### Container runtime

The runtime, such as containerd, pulls the image and creates the actual container.

Therefore:

```text
kubectl
   ↓
Pod object
   ↓
Scheduler
   ↓
Kubelet
   ↓
containerd
   ↓
Container
```

---

# 15. Pod Events

The `describe` command showed events similar to:

```text
Scheduled
Pulling image "nginx:latest"
Successfully pulled image "nginx:latest"
Created container
Started container
```

This is the normal Pod startup sequence.

Simplified:

```text
Scheduled
   ↓
Image Pull
   ↓
Container Created
   ↓
Container Started
   ↓
Pod Running
```

---

# 16. Why Does the Pod Have a Container Name?

When `kubectl run` is used without explicitly specifying a different container name, the generated Pod commonly uses the Pod name as the container name.

In this task:

```text
Pod:
dev-nginx-pod
```

The container appeared as:

```text
dev-nginx-pod
```

This is valid because the task did not require a separate container name.

---

# 17. Important Pod Statuses

## Pending

The Pod has not reached the Running state.

Possible causes:

* Insufficient CPU
* Insufficient memory
* Scheduling constraints
* Node unavailable
* Volume problems

Check:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

---

## Running

The Pod has been scheduled and its container is running.

However:

> Running does not automatically mean the application is ready to receive traffic.

Readiness probes are used for application readiness.

---

## CrashLoopBackOff

The container repeatedly starts and crashes.

Check:

```bash
kubectl logs <pod-name> -n <namespace>
```

Also:

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

And:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

---

## ImagePullBackOff

Kubernetes cannot successfully obtain the container image.

Check:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Possible causes:

* Incorrect image name
* Incorrect tag
* Registry authentication failure
* Registry/network issue
* Image does not exist

---

# 18. Pod Is Running but Application Is Not Working

Do not assume:

```text
Running = Application healthy
```

A better troubleshooting path is:

```text
Pod
 ↓
Container
 ↓
Application process
 ↓
Readiness probe
 ↓
Service
 ↓
Endpoints
 ↓
Ingress / Load Balancer
 ↓
Network
 ↓
DNS
```

Useful commands:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
```

---

# 19. Namespace Commands

List namespaces:

```bash
kubectl get namespaces
```

Short form:

```bash
kubectl get ns
```

Create namespace:

```bash
kubectl create namespace dev
```

Delete namespace:

```bash
kubectl delete namespace dev
```

Use a namespace:

```bash
kubectl get pods -n dev
```

List resources in all namespaces:

```bash
kubectl get pods -A
```

---

# 20. Pod Commands

List Pods:

```bash
kubectl get pods
```

List Pods in a namespace:

```bash
kubectl get pods -n dev
```

List Pods with node/IP information:

```bash
kubectl get pods -n dev -o wide
```

Describe a Pod:

```bash
kubectl describe pod dev-nginx-pod -n dev
```

View logs:

```bash
kubectl logs dev-nginx-pod -n dev
```

Execute a command:

```bash
kubectl exec -it dev-nginx-pod -n dev -- /bin/bash
```

Delete a Pod:

```bash
kubectl delete pod dev-nginx-pod -n dev
```

---

# 21. What Happens If This Pod Is Deleted?

This Pod was created directly using:

```bash
kubectl run
```

It is not managed by a Deployment.

Therefore, if you run:

```bash
kubectl delete pod d
ev-nginx-pod -n dev
```

Kubernetes does not automatically create a replacement Pod.

This is an important distinction.

```text
Standalone Pod
    ↓
Pod deleted
    ↓
No automatic replacement
```

Compare this with a Deployment:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Pod deleted
    ↓
ReplicaSet creates replacement
```

For production applications, Deployments are generally preferred over standalone Pods.

---

# 22. Pod YAML Equivalent

The same Pod can be represented as YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx-pod
  namespace: dev
spec:
  containers:
    - name: dev-nginx-pod
      image: nginx:latest
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Verify:

```bash
kubectl get pod dev-nginx-pod -n dev
```

---

# 23. Why Use YAML in Production?

Imperative command:

```bash
kubectl run dev-nginx-pod --image=nginx:latest -n dev
```

is convenient for labs and quick tests.

Declarative YAML:

```bash
kubectl apply -f pod.yaml
```

is better for:

* GitOps
* Version control
* Code review
* Reproducibility
* CI/CD
* Infrastructure documentation

---

# 24. Important Production Recommendation

The task requires:

```text
nginx:latest
```

For production, prefer a fixed version such as:

```text
nginx:1.27.2
```

or another approved immutable image reference.

Why avoid `latest`?

Because:

```text
Today:
nginx:latest → Image A

Later:
nginx:latest → Image B
```

The same deployment configuration could therefore result in different application binaries.

Versioned images provide:

* Predictable deployments
* Easier rollback
* Better auditing
* Reproducibility
* Safer CI/CD

---

# 25. L1 Interview Questions — Junior

## Q1. What is a Kubernetes Namespace?

A Namespace is a logical partition used to organize and control Kubernetes resources within a cluster.

Example:

```text
dev
test
prod
```

---

## Q2. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

It contains one or more containers.

---

## Q3. How do you create a Namespace?

```bash
kubectl create namespace dev
```

---

## Q4. How do you create a Pod?

```bash
kubectl run nginx --image=nginx:latest
```

---

## Q5. How do you create a Pod in a particular Namespace?

```bash
kubectl run nginx \
  --image=nginx:latest \
  -n dev
```

---

## Q6. How do you list Pods in a Namespace?

```bash
kubectl get pods -n dev
```

---

## Q7. How do you list Pods from all Namespaces?

```bash
kubectl get pods -A
```

---

# 26. L2 Interview Questions — Intermediate

## Q8. Can two Namespaces contain Pods with the same name?

Yes.

For example:

```text
dev/nginx
prod/nginx
```

These are different Kubernetes objects because the Namespace is part of their identity.

---

## Q9. Does Namespace provide complete isolation?

No.

Namespaces provide logical isolation.

Additional controls may include:

* RBAC
* NetworkPolicies
* ResourceQuotas
* LimitRanges
* Dedicated nodes
* Separate clusters

---

## Q10. What happens when a standalone Pod is deleted?

Kubernetes does not automatically recreate it.

A Deployment or another controller is normally required for automatic replacement.

---

## Q11. What is the difference between a Pod and a container?

A container runs the application process.

A Pod is the Kubernetes unit that hosts one or more containers and provides the shared networking/storage context for those containers.

---

## Q12. Why does a Pod get an IP?

Kubernetes networking assigns a Pod IP so workloads can communicate over the cluster network.

Pod IPs are generally ephemeral.

Applications normally use Services for stable access.

---

# 27. L3 Interview Questions — Senior

## Q13. A Pod is Pending. How do you troubleshoot?

Start with:

```bash
kubectl get pod <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl get nodes
kubectl get events --sort-by=.metadata.creationTimestamp
```

Look for:

* Insufficient resources
* Node problems
* Taints
* Affinity
* Scheduling failures
* Volume issues

The key principle is:

> Start with the Pod events because Kubernetes usually tells you why scheduling failed.

---

## Q14. Pod is CrashLoopBackOff. What do you check?

First:

```bash
kubectl logs <pod> -n <namespace>
```

Then:

```bash
kubectl logs <pod> -n <namespace> --previous
```

Then:

```bash
kubectl describe pod <pod> -n <namespace>
```

Investigate:

* Application crash
* Incorrect command
* Environment variables
* Secrets/configuration
* Permissions
* Probes
* Dependency failures

---

## Q15. Pod is Running but users receive HTTP 503. What do you check?

Use:

```bash
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl describe svc <service> -n <namespace>
kubectl get pods -n <namespace> --show-labels
```

Most common areas:

```text
Pod labels
    ↓
Service selecto
r
    ↓
Endpoints
    ↓
Target port
    ↓
Readiness
    ↓
Ingress / Load Balancer
```

---

# 28. L4 Interview Questions — Architect

## Q16. Would you use standalone Pods for a production microservice?

Generally no.

For stateless applications:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pods
```

A Deployment provides:

* Desired replica count
* Self-healing
* Rolling updates
* Rollbacks
* Version management

---

## Q17. How would you separate development and production?

One common model:

```text
Cluster
│
├── dev
├── test
├── staging
└── prod
```

But for strong isolation requirements, separate clusters/accounts may be more appropriate:

```text
AWS Account
│
├── Non-Production EKS
│
└── Production EKS
```

The decision depends on:

* Security
* Compliance
* Blast radius
* Cost
* Operational complexity
* Team ownership

---

## Q18. How would you design a highly available Kubernetes application?

Use:

* Multiple worker nodes
* Multiple Availability Zones
* Multiple Pod replicas
* Pod anti-affinity/topology spread
* Readiness probes
* Liveness probes where appropriate
* Horizontal Pod Autoscaler
* PodDisruptionBudget
* Load Balancer
* Multi-AZ database
* Centralized logging
* Metrics and alerting

Example:

```text
                Internet
                   |
              Load Balancer
                   |
                Service
                   |
          +--------+--------+
          |        |        |
         Pod      Pod      Pod
          |        |        |
        AZ-A     AZ-B     AZ-C
```

---

# 29. AWS EKS Mapping

The Kubernetes task maps naturally to Amazon EKS.

```text
Kubernetes
     |
     v
Amazon EKS
```

Common architecture:

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
Containers
```

Supporting AWS services:

| Requirement              | AWS Service                   |
| ------------------------ | ----------------------------- |
| Managed Kubernetes       | Amazon EKS                    |
| Worker compute           | EC2 / EKS managed node groups |
| Container images         | Amazon ECR                    |
| Networking               | Amazon VPC                    |
| Identity                 | IAM                           |
| Load balancing           | Elastic Load Balancing        |
| DNS                      | Route 53                      |
| Monitoring               | CloudWatch                    |
| Persistent block storage | EBS                           |
| Shared file storage      | EFS                           |

---

# 30. AWS Scenario — Pod Running but Cannot Reach AWS Service

### Scenario

A Pod runs successfully but cannot access an AWS service.

### Investigation

Inside the application environment, check:

```text
Pod
 ↓
DNS
 ↓
Route
 ↓
NAT / VPC endpoint
 ↓
Security controls
 ↓
AWS service
```

In EKS investigate:

* Pod networking
* Route tables
* NAT Gateway
* VPC endpoints
* Security Groups
* Network ACLs
* DNS
* IAM permissions

Important distinction:

```text
Network problem
```

and:

```text
IAM authorization problem
```

are different failure domains.

---

# 31. AWS Scenario — EKS Pod Cannot Pull Image

### Symptoms

```text
ImagePullBackOff
```

### Investigation

```bash
kubectl describe pod <pod> -n <namespace>
```

Check:

* ECR repository exists
* Image/tag exists
* Node has network access
* ECR permissions are correct
* IAM role is correct
* Registry authentication works

Typical flow:

```text
EKS Node
   |
   +-- Network
   |
   +-- IAM
   |
   v
Amazon ECR
   |
   v
Container Image
```

---

# 32. AWS Scenario — EKS Application Returns 503

### Symptoms

```text
Pods: Running
Service: Exists
Users: HTTP 503
```

Troubleshoot:

```bash
kubectl get pods -n <namespace> --show-labels
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl describe svc <service> -n <namespace>
```

Possible root causes:

* Service selector mismatch
* No healthy endpoints
* Readiness probe failure
* Wrong target port
* Load Balancer target health failure
* Security Group issue
* Ingress configuration error

---

# 33. Real AWS Incident RCA #1 — December 2021 US-EAST-1

## Incident

AWS experienced a major service event in the Northern Virginia region in December 2021.

AWS reported that an internal networking issue caused impact to multiple AWS service control planes.

### Important lesson

A running EC2 instance could remain healthy while APIs used to create or manage resources experienced errors or latency.

This demonstrates an important cloud architecture concept:

```text
Data Plane
     ≠
Control Plane
```

### Example

An existing workload may continue running while:

```text
Create EC2
Describe EC2
Provision Load Balancer
Change DNS
```

experience failures.

### Root Cause Category

AWS described the event as being triggered by automated scaling activity that produced a previously unobserved behavior in internal networking components, resulting in congestion.

### Impact

AWS reported impact to multiple services and their control planes.

### Corrective Actions

AWS described actions including:

* Disabling the triggering scaling activities
* Deploying remediations
* Additional network protection
* Improving backoff behavior
* Improving operational communication

### DevOps lesson

Do not design production systems assuming:

```text
One AWS Region
+
One control plane
=
Zero risk
```

Design for:

* Multi-AZ
* Appropriate multi-region capability
* Graceful degradation
* Cached configuration
* Retry with exponential backoff
* Circuit breakers
* Operational independence

---

# 34. Real AWS Incident RCA #2 — Kinesis Event, July 2024

AWS reported a Kinesis Data Streams event in US-EAST-1 in July 2024.

The incident affected some services that depended directly or indirectly on the affected Kinesis cell.

AWS explained that one internal Kinesis cell experienced an impairment associated with its cell-management system and an unusual workload profile.

A deployment caused hosts to be taken in and out of service. Work redistribution interacted poorly with a workload containing a very large number of low-throughput shards.

This led to oversized health/status messages, delayed processing, incorrect health determinations, excessive redistribution, and eventually resource contention.

### Simplified chain

```text
Routine deployment
       |
       v
Work redistribution
       |
       v
Unusual shard workload
       |
       v
Large status messages
       |
       v
Delayed health processing
       |
       v
Incorrect redistribution
       |
       v
Resource contention
       |
       v
Kinesis degradation
       |
       v
Dependent services impacted
```

### Services affected

AWS reported impact to services including:

* CloudWatch Logs
* Data Firehose
* S3 event delivery
* ECS
* Lambda
* Redshift
* Glue

### DevOps lesson

This is a classic example of:

> A dependency failure propagating through a distributed system.

For Kubernetes workloads, the same principle applies.

For example:

```text
Application
    ↓
Logging
    ↓
CloudWatch
    ↓
Dependency failure
    ↓
Application impact
```

Therefore, production systems should consider:

* Dependency timeouts
* Retries
* Backoff
* Circuit breakers
* Queue buffering
* Non-blocking logging where appropriate
* Graceful degradation

---

# 35. General RCA Template

When handling a production incident, structure the RCA as:

## Incident

What happened?

## Impact

Who/what was affected?

## Detection

How was the issue detected?

## Timeline

When did it start?

When was mitigation applied?

When was service restored?

## Root Cause

What technically caused the failure?

## Contributing Factors

What made the incident worse?

## Mitigation

What restored service?

## Corrective Actions

What prevents recurrence?

## Preventive Actions

What architectural changes reduce blast radius?

## Lessons Learned

What should engineers change?

---

# 36. Kubernetes RCA Example

## Incident

Nginx application unavailable.

## Symptoms

```text
Pods = Running
Service = Available
Endpoints = Empty
```

## Investigation

```bash
kubectl get pods --show-labels
kubectl get svc
kubectl get endpoints
```

Pod:

```yaml
labels:
  app: nginx
```

Service:

```yaml
selector:
  app: web
```

## Root Cause

The Service selector did not match the Pod labels.

Therefore:

```text
Service
   |
   | selector: app=web
   X
Pod
   |
   | label: app=nginx
```

No endpoint was created.

## Resolution

Correct the Service selector:

```yaml
selector:
  app: nginx
```

## Preventive Actions

* Validate selectors in CI/CD
* Use standard labels
* Add deployment smoke tests
* Monitor endpoint availability
* Monitor Load Balancer target health

---

# 37. Full DevOps Mock Interview

## Question 1

Create the required Pod.

### Answer

```bash
kubectl create namespace dev

kubectl run dev-nginx-pod \
  --image=nginx:latest \
  -n dev
```

Verify:

```bash
kubectl get pods -n dev
```

---

## Question 2

How do you prove that the image tag is correct?

### Answer

```bash
kubectl get pod dev-nginx-pod -n dev -o yaml | grep image
```

Expected:

```text
image: nginx:latest
```

---

## Question 3

What is the difference between a Pod and Deployment?

### Answer

A Pod is the smallest deployable unit.

A Deployment manages application Pods and provides:

* Replica management
* Rolling updates
* Rollback
* Self-healing through ReplicaSets

---

## Question 4

What happens if you delete this Pod?

### Answer

Because it is a standalone Pod:

```bash
kubectl delete pod dev-nginx-pod -n dev
```

Kubernetes will not automatically create another Pod.

If it were managed by a Deployment:

```text
Pod deleted
   ↓
ReplicaSet detects replica shortage
   ↓
Replacement Pod created
```

---

## Question 5

Why should production applications normally use Deployments?

### Answer

Deployments provide controlled application lifecycle management.

They support:

```text
Scaling
Rolling updates
Rollback
Self-healing
Replica management
```

A standalone Pod does not provide these capabilities.

---

## Question 6

Pod is Pending. What do you do?

### Answer

Run:

```bash
kubectl describe pod <pod> -n <namespace>
```

Then:

```bash
kubectl get nodes
kubectl get events --sort-by=.metadata.creationTimestamp
```

Investigate scheduler events, resources, taints, affinity, and node availability.

---

## Question 7

Pod is CrashLoopBackOff. What do you do?

### Answer

```bash
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl describe pod <pod> -n <namespace>
```

Investigate the application process, command, configuration, environment, permissions, dependencies, and probes.

---

## Question 8

Pod is Running but users cannot connect.

### Answer

Check:

```bash
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get pods -n <namespace> --show-labels
```

Then investigate:

```text
Pod
 ↓
Readiness
 ↓
Service selector
 ↓
Endpoints
 ↓
Ingress / Load Balancer
 ↓
Security Group
 ↓
Network
 ↓
DNS
```

---

## Question 9

How would you make the application highly available?

### Answer

Use:

* Multiple replicas
* Multiple Availability Zones
* Pod topology spread
* PodDisruptionBudget
* Readiness probes
* Horizontal Pod Autoscaler
* Load Balancer
* Multi-AZ data layer
* Monitoring and alerting

---

## Question 10

How would you deploy this application to AWS?

### Answer

Use Amazon EKS.

Architecture:

```text
Route 53
   ↓
AWS Load Balancer
   ↓
Kubernetes Service
   ↓
Deployment
   ↓
ReplicaSet
   ↓
Pods
   ↓
Containers
```

Images can be stored in Amazon ECR.

Worker nodes run inside an AWS VPC.

Monitoring can use CloudWatch and Kubernetes-native observability tools.

---

# 38. DevOps Troubleshooting Framework

When an incident occurs, avoid randomly executing commands.

Use a structured approach.

## Step 1 — Identify the symptom

Example:

```text
HTTP 503
```

## Step 2 — Determine the scope

Is it:

```text
One Pod?
One node?
One namespace?
One AZ?
Entire cluster?
Entire region?
```

## Step 3 — Check recent changes

Look for:

* Deployment
* Configuration
* Image
* Network change
* IAM change
* DNS change
* Infrastructure change

## Step 4 — Follow the request path

```text
Client
 ↓
DNS
 ↓
Load Balancer
 ↓
Ingress
 ↓
Service
 ↓
Endpoint
 ↓
Pod
 ↓
Container
 ↓
Application
 ↓
Database / External dependency
```

## Step 5 — Establish root cause

Do not stop at:

```text
Pod restarted
```

Ask:

```text
Why did the Pod restart?
```

Then:

```text
Why did that happen?
```

Continue until the technical root cause is identified.

---

# 39. Important Production Commands

## Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl describe node <node>
```

## Namespace

```bash
kubectl get ns
kubectl get all -n dev
```

## Pods

```bash
kubectl get pods -n dev
kubectl get pods -n dev -o wide
kubectl describe pod <pod> -n dev
kubectl logs <pod> -n dev
kubectl logs <pod> -n dev --previous
```

## Events

```bash
kubectl get events -n dev --sort-by=.metadata.creationTimestamp
```

## Services

```bash
kubectl get svc -n dev
kubectl describe svc <service> -n dev
kubectl get endpoints -n dev
```

---

# 40. Most Important Kubernetes Concepts to Remember

```text
Cluster
   ↓
Namespace
   ↓
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
```

For this particular task:

```text
Cluster
   ↓
dev Namespace
   ↓
dev-nginx-pod
   ↓
nginx container
   ↓
nginx:latest
```

For a production application:

```text
Cluster
   ↓
Namespace
   ↓
Deployment
   ↓
ReplicaSet
   ↓
Multiple Pods
   ↓
Containers
```

---

# 41. Final Task Validation

Run:

```bash
kubectl get namespace dev
```

Expected:

```text
NAME   STATUS
dev    Active
```

Run:

```bash
kubectl get pod dev-nginx-pod -n dev
```

Expected:

```text
NAME             READY   STATUS    RESTARTS
dev-nginx-pod    1/1     Running   0
```

Run:

```bash
kubectl get pod dev-nginx-pod -n dev -o yaml | grep image
```

Expected:

```text
image: nginx:latest
```

Run:

```bash
kubectl describe pod dev-nginx-pod -n dev
```

Confirm:

```text
Namespace: dev
Status: Running
Image: nginx:latest
Ready: True
Restart Count: 0
```

---

# 42. Final Answer for the Lab

The exact commands required are:

```bash
kubectl create namespace dev

kubectl run dev-nginx-pod \
  --image=nginx:latest \
  --namespace=dev
```

Validation:

```bash
kubectl get pods -n dev

kubectl get pod dev-nginx-pod \
  -n dev \
  -o yaml | grep image
```

Expected:

```text
dev-nginx-pod   1/1   Running   0
```

and:

```text
image: nginx:latest
```

Therefore the task is successfully completed.

---

# 43. Quick Interview Revision

## Namespace

```text
Logical partition inside a Kubernetes cluster.
```

## Pod

```text
Smallest deployable Kubernetes unit.
```

## Container

```text
Runs the application process.
```

## Kubelet

```text
Node agent responsible for managing Pods on a worker node.
```

## Container Runtime

```text
Actually creates and runs containers.
Example: containerd
```

## Deployment

```text
Manages application Pods and provides rolling updates, scaling, and rollback.
```

## ReplicaSet

```text
Maintains the desired number of Pods.
```

## Service

```text
Provides stable networking/access to Pods.
```

---

# 44. Golden Troubleshooting Rules

### Rule 1

Never assume:

```text
Pod Running = Application Healthy
```

### Rule 2

For `Pending`:

```bash
kubectl describe pod
```

### Rule 3

For `CrashLoopBackOff`:

```bash
kubectl logs
kubectl logs --previous
```

### Rule 4

For networking problems:

```text
Pod → Service → Endpoint → Ingress/LB → Network
```

### Rule 5

For image problems:

```bash
kubectl describe pod
```

### Rule 6

For production applications:

```text
Prefer Deployment over standalone Pod.
```

### Rule 7

For production images:

```text
Prefer immutable versioned images over latest.
```

### Rule 8

During an incident:

```text
Symptom
 ↓
Scope
 ↓
Evidence
 ↓
Recent Change
 ↓
Root Cause
 ↓
Mitigation
 ↓
Corrective Action
```

---

# 45. Final Mental Model

The most important concept from this task is the complete lifecycle:

```text
User
 |
 | kubectl run
 v
Kubernetes API Server
 |
 v
Pod Object
 |
 v
Scheduler
 |
 v
Worker Node
 |
 v
Kubelet
 |
 v
Container Runtime
 |
 v
Container
 |
 v
Nginx Application
```

And the namespace relationship:

```text
Kubernetes Cluster
       |
       v
    Namespace
       |
       v
      Pod
       |
       v
   Container
       |
       v
 Application
```

For production:

```text
Kubernetes Cluster
       |
       v
    Namespace
       |
       v
   Deployment
       |
       v
   ReplicaSet
       |
       v
 Multiple Pods
       |
       v
   Containers
```

This distinction between **Namespace → Pod → Container** and **Deployment → ReplicaSet → Pod** is one of the most important Kubernetes fundamentals for both real-world DevOps work and interviews.

