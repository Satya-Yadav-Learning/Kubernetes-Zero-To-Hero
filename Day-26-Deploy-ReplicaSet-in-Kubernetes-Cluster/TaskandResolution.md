# Kubernetes ReplicaSet Task — Task and Resolution

## 1. Task Overview

### Objective

Create a Kubernetes ReplicaSet with the following requirements:

| Requirement | Value |
|---|---|
| Resource type | ReplicaSet |
| ReplicaSet name | `httpd-replicaset` |
| Container name | `httpd-container` |
| Image | `httpd:latest` |
| Replicas | `4` |
| Application label | `app=httpd_app` |
| Tier label | `type=front-end` |

### Expected Result

The ReplicaSet must maintain exactly 4 Pods using the `httpd:latest` image.

---

# 2. Final Kubernetes Manifest

File:

`httpd-replicaset.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset
  labels:
    app: httpd_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      app: httpd_app
      type: front-end
  template:
    metadata:
      labels:
        app: httpd_app
        type: front-end
    spec:
      containers:
        - name: httpd-container
          image: httpd:latest
```

---

# 3. Manifest Explanation

## apiVersion

```yaml
apiVersion: apps/v1
```

The ReplicaSet API belongs to the `apps/v1` API group.

---

## kind

```yaml
kind: ReplicaSet
```

Defines the Kubernetes resource type.

A ReplicaSet is a Kubernetes controller responsible for maintaining the desired number of matching Pods.

---

## metadata.name

```yaml
metadata:
  name: httpd-replicaset
```

Defines the name of the ReplicaSet.

---

## metadata.labels

```yaml
labels:
  app: httpd_app
  type: front-end
```

These labels identify the ReplicaSet itself.

They are useful for organization, filtering and management.

---

# 4. Replica Count

```yaml
spec:
  replicas: 4
```

This tells the ReplicaSet controller:

> Maintain 4 matching Pods.

If the desired number is 4 and only 3 Pods are running, the ReplicaSet attempts to create another Pod.

If 5 matching Pods exist, the controller attempts to reduce the number back to 4.

This is an example of Kubernetes desired-state reconciliation.

---

# 5. Selector

```yaml
selector:
  matchLabels:
    app: httpd_app
    type: front-end
```

The selector tells the ReplicaSet which Pods belong to it.

The ReplicaSet looks for Pods matching:

```text
app=httpd_app
type=front-end
```

The selector must match the labels defined in the Pod template.

---

# 6. Pod Template

```yaml
template:
  metadata:
    labels:
      app: httpd_app
      type: front-end
```

These labels are extremely important.

The ReplicaSet selector is:

```text
app=httpd_app
type=front-end
```

The Pod template contains exactly the same labels:

```text
app=httpd_app
type=front-end
```

Therefore, the ReplicaSet can correctly identify and manage the Pods it creates.

---

# 7. Container Configuration

```yaml
containers:
  - name: httpd-container
    image: httpd:latest
```

The Pod contains one container:

```text
Container name = httpd-container
Image = httpd:latest
```

The `httpd` image provides the Apache HTTP Server.

---

# 8. Validation Before Deployment

The manifest can be checked using client-side dry run:

```bash
kubectl apply --dry-run=client -f httpd-replicaset.yaml
```

Successful validation output:

```text
replicaset.apps/httpd-replicaset created (dry run)
```

## What does dry run mean?

The command validates the manifest locally from the client's perspective but does not actually create the ReplicaSet in the cluster.

Important distinction:

```text
--dry-run=client
        |
        v
Client-side validation
        |
        v
No actual Kubernetes resource created
```

The real deployment requires:

```bash
kubectl apply -f httpd-replicaset.yaml
```

---

# 9. Create the ReplicaSet

Final successful command:

```bash
kubectl apply -f httpd-replicaset.yaml
```

Output:

```text
replicaset.apps/httpd-replicaset created
```

This confirms that the ReplicaSet was successfully created.

---

# 10. Verify ReplicaSet

Command:

```bash
kubectl get replicaset httpd-replicaset
```

Output:

```text
NAME               DESIRED   CURRENT   READY   AGE
httpd-replicaset   4         4         4       23s
```

## Interpretation

```text
DESIRED = 4
CURRENT = 4
READY   = 4
```

This means:

- Kubernetes wants 4 Pods.
- 4 Pods currently exist.
- All 4 Pods are Ready.

Therefore, the ReplicaSet is operating successfully.

---

# 11. Verify Pods Using Labels

Command:

```bash
kubectl get pods -l app=httpd_app,type=front-end
```

Output:

```text
NAME                     READY   STATUS    RESTARTS   AGE
httpd-replicaset-dw964   1/1     Running   0          38s
httpd-replicaset-fzhk2   1/1     Running   0          38s
httpd-replicaset-qdfxr   1/1     Running   0          38s
httpd-replicaset-svkgz   1/1     Running   0          38s
```

## Interpretation

There are four matching Pods.

Each shows:

```text
READY = 1/1
STATUS = Running
RESTARTS = 0
```

Therefore, all four Pods are healthy from the basic Kubernetes readiness perspective.

---

# 12. Complete Verification Flow

The recommended verification sequence is:

```bash
kubectl get replicaset httpd-replicaset
```

Then:

```bash
kubectl get pods -l app=httpd_app,type=front-end
```

Then, when deeper troubleshooting is required:

```bash
kubectl describe rs httpd-replicaset
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

# 13. Kubernetes Architecture Flow

The complete flow is:

```text
kubectl
   |
   v
Kubernetes API Server
   |
   v
ReplicaSet Object
   |
   v
ReplicaSet Controller
   |
   +-------------------+
   |        |          |
   v        v          v
 Pod 1    Pod 2      Pod 3 ... Pod 4
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
httpd Container
```

Important:

> A ReplicaSet does not directly run the container.

The ReplicaSet creates/manages Pods.

The scheduler selects an appropriate node.

The kubelet on that node ensures the Pod is running.

The container runtime starts the container.

---

# 14. Desired State vs Actual State

Kubernetes works primarily through desired-state reconciliation.

For this task:

```text
Desired state:
4 Pods
```

Suppose the actual state becomes:

```text
3 Pods
```

The ReplicaSet controller detects the difference:

```text
Desired = 4
Actual  = 3
Difference = 1
```

The controller creates another Pod.

Eventually:

```text
Desired = 4
Actual  = 4
```

This is called reconciliation.

---

# 15. Self-Healing Scenario

Suppose one of the four Pods is deleted:

```bash
kubectl delete pod <pod-name>
```

The ReplicaSet notices that the number of Pods has dropped.

Before deletion:

```text
Desired = 4
Current = 4
```

After deletion:

```text
Desired = 4
Current = 3
```

The ReplicaSet controller creates a replacement Pod.

Eventually:

```text
Desired = 4
Current = 4
Ready   = 4
```

This demonstrates Kubernetes self-healing.

---

# 16. Important Difference: ReplicaSet vs Pod

A Pod is the smallest deployable unit in Kubernetes.

A ReplicaSet is a controller.

```text
Pod
|
+-- Runs one or more containers

ReplicaSet
|
+-- Maintains the desired number of Pods
```

The ReplicaSet provides:

- Replica management
- Self-healing
- Desired-state reconciliation
- Pod replacement

---

# 17. ReplicaSet vs Deployment

In production, a Deployment is normally preferred over directly managing a ReplicaSet.

Architecture:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    v
Containers
```

A Deployment provides additional capabilities such as:

- Rolling updates
- Rollbacks
- Revision history
- Controlled replacement of ReplicaSets

A directly created ReplicaSet does not provide the same deployment-management capabilities.

Interview answer:

> "ReplicaSet maintains the desired number of Pods, while Deployment manages ReplicaSets and provides application rollout, update and rollback capabilities. For production workloads I would normally use a Deployment rather than managing a ReplicaSet directly."

---

# 18. Important Production Concern: httpd:latest

The task specifically requires:

```yaml
image: httpd:latest
```

This is acceptable for the lab requirement.

However, using `latest` is generally not recommended for production.

Why?

The `latest` tag is mutable.

The same tag can point to different image versions over time.

For production, use a versioned tag such as:

```yaml
image: httpd:2.4
```

Even stronger immutability can be achieved with an image digest:

```yaml
image: httpd@sha256:<image-digest>
```

Benefits:

- Reproducible deployments
- Predictable releases
- Safer rollbacks
- Better auditability
- Reduced configuration drift

---

# 19. Useful Kubernetes Commands

## List ReplicaSets

```bash
kubectl get rs
```

## Get a specific ReplicaSet

```bash
kubectl get rs httpd-replicaset
```

## Detailed ReplicaSet information

```bash
kubectl describe rs httpd-replicaset
```

## List Pods

```bash
kubectl get pods
```

## Filter Pods by labels

```bash
kubectl get pods -l app=httpd_app,type=front-end
```

## Describe a Pod

```bash
kubectl describe pod <pod-name>
```

## View Pod logs

```bash
kubectl logs <pod-name>
```

## View cluster events

```bash
kubectl get events --sort-by=.lastTimestamp
```

## Verify the image

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[0].image}'
```

Expected:

```text
httpd:latest
```

---

# 20. Scaling the ReplicaSet

The current ReplicaSet has:

```text
replicas = 4
```

It can be scaled:

```bash
kubectl scale replicaset httpd-replicaset --replicas=6
```

Verify:

```bash
kubectl get rs httpd-replicaset
```

Expected:

```text
DESIRED = 6
```

Then verify Pods:

```bash
kubectl get pods -l app=httpd_app,type=front-end
```

---

# 21. Troubleshooting Scenarios

## Scenario 1: DESIRED 4, CURRENT 4, READY 2

Example:

```text
NAME               DESIRED   CURRENT   READY
httpd-replicaset   4         4         2
```

Interpretation:

- Four Pods exist.
- Only two are Ready.
- The ReplicaSet itself may be functioning correctly.
- The problem is likely with individual Pods.

Investigate:

```bash
kubectl get pods -l app=httpd_app,type=front-end
```

Then:

```bash
kubectl describe pod <pod-name>
```

Then:

```bash
kubectl logs <pod-name>
```

Then:

```bash
kubectl get events --sort-by=.lastTimestamp
```

Possible causes:

- Container startup failure
- Readiness probe failure
- Resource issue
- Image issue
- Configuration issue
- Node problem

---

# 22. Scenario 2: Pod Stuck in Pending

Check:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

Look at Events.

Potential causes:

- Insufficient CPU
- Insufficient memory
- Node unavailable
- Taints
- Missing tolerations
- Affinity rules
- Scheduling constraints
- Resource quotas

Interview answer:

> "For a Pending Pod, I would first describe the Pod and inspect the scheduler events. I would then check node capacity, taints, tolerations, affinity, resource requests and namespace quotas."

---

# 23. Scenario 3: ImagePullBackOff

Check:

```bash
kubectl describe pod <pod-name>
```

Then inspect:

```text
Events
```

Potential causes:

- Incorrect image name
- Incorrect image tag
- Registry unavailable
- Authentication problem
- Network connectivity problem
- Registry permissions

For a private registry, also verify registry credentials and permissions.

---

# 24. Scenario 4: CrashLoopBackOff

Start with:

```bash
kubectl logs <pod-name>
```

If the container restarted:

```bash
kubectl logs <pod-name> --previous
```

Then:

```bash
kubectl describe pod <pod-name>
```

Investigate:

- Application startup error
- Configuration
- Environment variables
- Secrets
- ConfigMaps
- Probes
- Resource limits
- Dependency failures
- Exit codes

Interview answer:

> "CrashLoopBackOff means Kubernetes is repeatedly starting the container and the container is repeatedly failing. I would inspect current and previous logs, Pod events, exit codes, probes, configuration and resource constraints."

---

# 25. Scenario 5: ReplicaSet Has No Pods

Check:

```bash
kubectl describe rs httpd-replicaset
```

Check:

```bash
kubectl get pods -l app=httpd_app,type=front-end
```

Then:

```bash
kubectl get events --sort-by=.lastTimestamp
```

Possible areas:

- Selector mismatch
- Pod creation failures
- Admission policy
- Resource quota
- Scheduling failure
- Node capacity
- Image problems

One critical concept:

> ReplicaSet selectors and Pod template labels must match.

---

# 26. Scenario 6: Pod Is Running but Application Is Not Reachable

`Running` does not automatically mean the application is reachable.

Investigate:

```bash
kubectl get pods
```

```bash
kubectl describe pod <pod-name>
```

Then verify whether a Kubernetes Service exists:

```bash
kubectl get svc
```

If applicable:

```bash
kubectl describe svc <service-name>
```

Check:

- Pod labels
- Service selector
- Target port
- Container port
- Readiness
- Network policies
- Load balancer
- DNS

---

# 27. Important Interview Concept: Running vs Ready

A Pod can be:

```text
Running
```

but not:

```text
Ready
```

`Running` indicates that the Pod has been started and its containers are running.

`Ready` indicates that the Pod is considered ready to receive traffic according to its readiness conditions.

Therefore:

```text
Running != necessarily Ready
```

This distinction is important during production troubleshooting.

---

# 28. L1 Interview Questions — Junior Level

## Q1. What is a ReplicaSet?

Answer:

> A ReplicaSet is a Kubernetes controller that maintains a specified number of identical Pods. It continuously compares the desired number of replicas with the actual number and creates or removes Pods as necessary.

---

## Q2. Why did we configure replicas as 4?

Answer:

> The task requires four application Pods. The ReplicaSet controller continuously attempts to maintain four matching Pods.

---

## Q3. What is a label?

Answer:

> A label is a key-value metadata pair attached to Kubernetes objects. Labels are commonly used for identification, grouping and selection.

Example:

```text
app=httpd_app
type=front-end
```

---

## Q4. What is a selector?

Answer:

> A selector defines which Kubernetes objects a controller or Service should operate on or target.

In this task:

```yaml
matchLabels:
  app: httpd_app
  type: front-end
```

---

## Q5. Why must the labels match?

Answer:

> The ReplicaSet uses the selector to identify its Pods. Therefore, the Pod template labels must satisfy the ReplicaSet selector.

---

## Q6. What does `kubectl apply` do?

Answer:

> `kubectl apply` sends the desired configuration to the Kubernetes API server, which validates and stores the resource configuration. Kubernetes controllers then reconcile the desired state.

---

## Q7. What does `--dry-run=client` do?

Answer:

> It performs client-side validation without actually creating the Kubernetes resource.

---

# 29. L2 Interview Questions — Intermediate Level

## Q8. How does ReplicaSet maintain four Pods?

Answer:

> The ReplicaSet controller continuously reconciles the desired state with the actual state. If the desired replica count is four and only three matching Pods exist, the controller creates another Pod. If five exist, it removes excess Pods until the desired state is restored.

---

## Q9. What happens if a Pod is deleted?

Answer:

> The ReplicaSet detects that the actual replica count has fallen below the desired count and creates a replacement Pod.

---

## Q10. Does ReplicaSet create containers directly?

Answer:

> No. The ReplicaSet creates Pod objects. The scheduler assigns Pods to nodes, and the kubelet works with the container runtime to start the containers.

---

## Q11. What happens after `kubectl apply`?

Answer:

```text
kubectl
  |
  v
API Server
  |
  v
ReplicaSet stored
  |
  v
ReplicaSet Controller
  |
  v
Pod objects created
  |
  v
Scheduler assigns nodes
  |
  v
Kubelet starts containers
```

---

## Q12. How would you troubleshoot a ReplicaSet?

Answer:

```bash
kubectl get rs
kubectl describe rs httpd-replicaset
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.lastTimestamp
```

I would first determine whether the problem is:

- ReplicaSet reconciliation
- Pod creation
- Scheduling
- Image retrieval
- Container startup
- Application readiness
- Node health

---

# 30. L3 Interview Questions — Senior Level

## Q13. Why would you use Deployment instead of ReplicaSet directly?

Answer:

> ReplicaSet provides replica management, but Deployment provides higher-level application lifecycle management including rolling updates, revision history and rollback. For production applications I would normally use a Deployment, which creates and manages ReplicaSets.

---

## Q14. How would you achieve high availability?

Answer:

I would consider:

- Multiple worker nodes
- Multiple Availability Zones
- Pod topology spread constraints
- Pod anti-affinity where appropriate
- PodDisruptionBudget
- Readiness probes
- Adequate resource capacity
- Cluster autoscaling
- Application-level redundancy
- Load balancing

The important point is that simply running four Pods does not automatically guarantee high availability.

If all four Pods land on one worker node and that node fails, all four may be affected.

---

## Q15. How would you prevent all replicas from landing on one node?

Answer:

Use scheduling controls such as:

- Topology spread constraints
- Pod anti-affinity
- Appropriate node labels
- Taints and tolerations where required

For multi-AZ environments, distribute replicas across failure domains.

---

## Q16. How would you safely update the application?

Answer:

I would use a Deployment and perform a controlled rolling update rather than directly changing a production ReplicaSet.

For example:

```bash
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
```

Then:

```bash
kubectl rollout status deployment/<deployment-name>
```

I would monitor:

- Pod readiness
- Error rate
- Latency
- Application health
- Resource utilization
- Logs
- Events

---

## Q17. How would you rollback?

Useful commands:

```bash
kubectl rollout history deployment/<deployment-name>
```

```bash
kubectl rollout undo deployment/<deployment-name>
```

```bash
kubectl rollout status deployment/<deployment-name>
```

The objective is to quickly return to a known-good application revision.

---

# 31. L4 Interview Questions — Architect Level

## Q18. Design a production architecture for this application.

A reasonable AWS architecture:

```text
                    Route 53
                       |
                       v
              AWS Load Balancer
                       |
                       v
              Kubernetes Service
                       |
                       v
                  Deployment
                       |
                       v
                 ReplicaSets
                       |
          +------------+------------+
          |            |            |
          v            v            v
       Pod/AZ-A     Pod/AZ-B     Pod/AZ-C
          |            |            |
          v            v            v
      httpd         httpd         httpd
```

Supporting services:

```text
Amazon ECR
     |
     v
Container Images

Amazon CloudWatch
     |
     v
Logs / Metrics / Alarms

IAM / Pod Identity
     |
     v
AWS Resource Access

VPC
     |
     +-- Subnets
     +-- Security Groups
     +-- Routing
     +-- Availability Zones
```

---

# 32. AWS Mapping

| Kubernetes / Requirement | AWS Equivalent or Integration |
|---|---|
| Kubernetes cluster | Amazon EKS |
| Container image registry | Amazon ECR |
| Load balancing | AWS Load Balancer integration |
| DNS | Amazon Route 53 |
| Monitoring | Amazon CloudWatch |
| Identity | IAM / EKS Pod Identity |
| Networking | Amazon VPC |
| Secrets | AWS Secrets Manager / Kubernetes Secrets |
| Object storage | Amazon S3 |
| Relational database | Amazon RDS |

Important:

> EKS provides managed Kubernetes capabilities, but ReplicaSet remains a native Kubernetes resource and behaves according to Kubernetes controller logic.

---

# 33. AWS Scenario Interview Question

## Q19. Your Kubernetes application is running on EKS, but users receive intermittent 5xx errors. How do you investigate?

Answer:

I would troubleshoot from the outside inward.

### Step 1 — DNS

Check Route 53 resolution.

### Step 2 — Load Balancer

Check:

- Target health
- 4xx/5xx metrics
- Connection errors
- Listener configuration
- Security groups

### Step 3 — Kubernetes Service

Check:

```bash
kubectl get svc
```

```bash
kubectl describe svc <service-name>
```

### Step 4 — Endpoints

Verify that the Service has healthy endpoints.

### Step 5 — Pods

```bash
kubectl get pods
```

Check:

- Ready status
- Restarts
- CPU
- Memory
- Logs

### Step 6 — Kubernetes Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

### Step 7 — AWS Monitoring

Check CloudWatch metrics and logs.

### Step 8 — Dependency Layer

Check:

- Database
- External APIs
- AWS services
- DNS
- Network connectivity

The goal is to correlate the user-facing error with the first failing component rather than immediately restarting Pods.

---

# 34. AWS Scenario: Pods Cannot Pull Image

Suppose Pods show:

```text
ImagePullBackOff
```

I would check:

```bash
kubectl describe pod <pod-name>
```

Then verify:

- Image name
- Image tag
- ECR repository
- ECR image availability
- Node/pod identity permissions
- Network connectivity
- Registry authentication

For EKS, IAM permissions and the mechanism used by the node or workload to access ECR must be verified.

---

# 35. AWS Scenario: Node Failure

Suppose multiple Pods disappear because a worker node fails.

A resilient architecture should:

- Run multiple worker nodes
- Use multiple Availability Zones
- Spread Pods across failure domains
- Maintain sufficient spare capacity
- Use autoscaling where appropriate
- Ensure workloads can be rescheduled

ReplicaSet can recreate Pods, but scheduling and available cluster capacity determine whether replacement Pods can actually run.

---

# 36. AWS Scenario: AZ Failure

Suppose an entire Availability Zone becomes unavailable.

If all replicas are located in that AZ:

```text
AZ-A
 |
 +-- Pod 1
 +-- Pod 2
 +-- Pod 3
 +-- Pod 4
```

the application can experience a major outage.

Better architecture:

```text
AZ-A       AZ-B       AZ-C
 |          |          |
Pod        Pod        Pod
 |          |          |
Pod        Pod        Pod
```

This reduces the blast radius of an AZ failure.

---

# 37. Real AWS Incident RCA — S3, 2017

## Incident

Amazon S3 experienced a significant service disruption in the US-EAST-1 Region on February 28, 2017.

## Root Cause

An authorized S3 team member was executing an established procedure intended to remove a small number of servers.

An incorrect input caused a larger set of servers to be removed than intended.

This affected important S3 subsystems and resulted in widespread service impact.

## Key Lessons

### 1. Blast radius

A maintenance operation intended for a small component affected a much larger system.

### 2. Guardrails

Critical operations need strong validation and safety mechanisms.

### 3. Automation

Automation should reduce human error rather than amplify it.

### 4. Incremental changes

Large changes should be broken into smaller, independently validated operations.

### Interview takeaway

> "One of the most important lessons from large-scale incidents is blast-radius control. Even authorized and well-understood operational procedures need guardrails, validation and incremental execution."

---

# 38. Real AWS Incident RCA — AWS Network Event, 2021

## Incident

AWS experienced a major networking event in the US-EAST-1 Region on December 7, 2021.

## Root Cause

Automated scaling activity triggered unexpected behavior and a large increase in connection activity from internal network clients.

Networking devices became overwhelmed.

This resulted in latency and errors.

Retries and additional connection activity contributed to further congestion.

## Key Lessons

### Retry storms

Retries can amplify an outage.

For distributed applications, retries should use:

- Exponential backoff
- Jitter
- Timeouts
- Rate limiting
- Circuit breakers

### Capacity

Capacity is not simply CPU and memory.

Network connection capacity and control-plane/network-device limits can also become bottlenecks.

### Scaling safety

Scaling events themselves need to be tested and monitored.

### Interview takeaway

> "A distributed system can turn a partial failure into a larger outage when clients retry aggressively. I would design retries with exponential backoff and jitter, enforce timeouts and rate limits, and monitor dependency saturation."

---

# 39. Real AWS Incident RCA — Amazon Kinesis, 2024

## Incident

Amazon Kinesis Data Streams experienced an availability event in US-EAST-1 on July 30, 2024.

## Root Cause

One internal Kinesis cell experienced an unusual workload containing a large number of very low-throughput shards.

During a routine deployment, hosts were taken in and out of service.

The cell-management system redistributed workloads in a way that caused some hosts to receive a very large number of shards.

This produced resource contention and increased latency/errors.

## Key Lessons

### Unusual workloads matter

Average workload testing is not enough.

Systems should be tested against:

- High-volume workloads
- Low-volume/high-object-count workloads
- Large numbers of idle or low-throughput resources
- Skewed distributions

### Distribution matters

Even when total capacity appears sufficient, uneven workload placement can create hotspots.

### Failure-domain isolation

Cells and partitions can reduce blast radius.

### Interview takeaway

> "Capacity planning should consider not only aggregate throughput but also workload distribution and resource cardinality. A system can fail because of an unusual concentration of small workloads even when average utilization looks healthy."

---

# 40. Real AWS Incident RCA — AWS Lambda, 2023

## Incident

AWS Lambda experienced an availability event in US-EAST-1 on June 13, 2023.

## Root Cause

A scaling transition crossed an unobserved capacity threshold and exposed a latent software defect.

The issue affected provisioning of underlying compute capacity and resulted in increased errors and latency.

Dependent services were also affected.

## Key Lessons

### Scaling transitions need testing

It is not enough to test steady-state operation.

Test:

```text
Normal load
     |
     v
Increasing load
     |
     v
Rapid scale-out
     |
     v
Capacity threshold
```

### Dependency observability

Monitor not only your application but also the dependencies underneath it.

### Capacity thresholds

Hidden thresholds are dangerous in distributed systems.

### Interview takeaway

> "I would test both steady-state and transition-state behavior, especially around scaling thresholds, and ensure that dependency capacity and saturation are observable."

---

# 41. Common AWS Incident Patterns

Across large distributed-system incidents, several recurring patterns appear.

## 1. Blast Radius

A small change should not be capable of taking down an entire platform.

Controls:

- Cells
- Isolation
- Canary deployments
- Progressive rollout
- Small batches

---

## 2. Retry Amplification

A dependency becomes slow.

Clients retry.

More requests arrive.

The dependency becomes even slower.

```text
Failure
  |
  v
Timeout
  |
  v
Retry
  |
  v
More traffic
  |
  v
More overload
  |
  v
Larger failure
```

Controls:

- Exponential backoff
- Jitter
- Timeouts
- Circuit breakers
- Rate limits

---

## 3. Capacity Exhaustion

Capacity can include:

- CPU
- Memory
- Disk
- Network
- Connections
- Threads
- File descriptors
- API limits
- Control-plane resources

---

## 4. Configuration Error

A configuration change can affect many systems simultaneously.

Controls:

- Validation
- Peer review
- Automated testing
- Canary deployment
- Rollback
- Policy controls

---

# 42. Production Deployment Strategy

For a production application, a safer model is:

```text
Developer
   |
   v
Git
   |
   v
CI Pipeline
   |
   +-- Test
   +-- Security Scan
   +-- Build
   +-- Image Scan
   |
   v
Amazon ECR
   |
   v
CD / GitOps
   |
   v
EKS Deployment
   |
   v
ReplicaSet
   |
   v
Pods
```

---

# 43. Production Readiness Checklist

A production Kubernetes workload should typically consider:

## Image

- Versioned image
- Immutable digest where appropriate
- Image scanning
- Trusted registry

## Resources

Define:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Actual values should be based on workload measurements.

## Health Checks

Consider:

- Readiness probe
- Liveness probe
- Startup probe

## Security

Consider:

- Non-root container
- Security context
- Least privilege
- NetworkPolicy
- Secret management
- Image scanning

## Availability

Consider:

- Multiple nodes
- Multiple AZs
- Topology spread
- PodDisruptionBudget
- Autoscaling

## Observability

Monitor:

- Logs
- Metrics
- Traces
- Error rate
- Latency
- Availability
- Resource utilization

---

# 44. Kubernetes Incident Response Methodology

When production breaks, use a structured process.

## Step 1 — Establish impact

Determine:

- Which users are affected?
- Which service is affected?
- Which region/AZ?
- How severe is the impact?

## Step 2 — Establish timeline

Determine:

- When did it start?
- What changed immediately before it?
- Did traffic increase?
- Did infrastructure change?

## Step 3 — Stabilize

Prioritize service recovery.

Potential actions:

- Stop rollout
- Roll back
- Scale capacity
- Remove unhealthy workload
- Fail over
- Disable problematic feature

## Step 4 — Identify root cause

Separate:

```text
Trigger
```

from:

```text
Root cause
```

and:

```text
Contributing factors
```

## Step 5 — Prevent recurrence

Create:

- Code fixes
- Configuration fixes
- Monitoring
- Alerts
- Automation
- Tests
- Runbooks
- Architecture changes

---

# 45. RCA Template

```text
Incident:
<incident name>

Date:
<date>

Impact:
<customer/business impact>

Detection:
<how incident was detected>

Timeline:
<important events>

Trigger:
<event that started the incident>

Root Cause:
<technical root cause>

Contributing Factors:
<additional factors>

Mitigation:
<how service was stabilized>

Recovery:
<how service was restored>

Corrective Actions:
<immediate engineering fixes>

Preventive Actions:
<long-term engineering/process changes>

Lessons Learned:
<key lessons>

Owner:
<team>

Status:
<open/closed>
```

---

# 46. MTTR and MTBF

## MTTR

Mean Time To Recovery/Repair.

It measures how quickly a service is restored after a failure.

Lower MTTR is generally better.

## MTBF

Mean Time Between Failures.

It measures the average time between failures.

A mature engineering organization aims to:

```text
Reduce failure frequency
+
Reduce recovery time
```

---

# 47. Full DevOps Mock Interview

## Question 1

### Interviewer

What did you implement in this task?

### Answer

> I created a Kubernetes ReplicaSet named `httpd-replicaset` using the `httpd:latest` image. I configured four replicas and used the labels `app=httpd_app` and `type=front-end`. I verified the ReplicaSet with `kubectl get replicaset` and verified that all four Pods were Running and Ready.

---

# 48. Question 2

### Interviewer

Why do you need four replicas?

### Answer

> Four replicas provide four instances of the application workload and allow the ReplicaSet to maintain four Pods continuously. If one Pod is deleted or fails, the ReplicaSet attempts to create a replacement.

---

# 49. Question 3

### Interviewer

What happens when one Pod crashes?

### Answer

> The ReplicaSet controller compares the desired replica count with the actual number of matching Pods. If the number drops below four, it creates a replacement Pod. However, if the application container itself repeatedly crashes, I would investigate the Pod logs, events, configuration and resource constraints.

---

# 50. Question 4

### Interviewer

What is the difference between ReplicaSet and Deployment?

### Answer

> ReplicaSet maintains the desired number of Pods. Deployment is a higher-level controller that manages ReplicaSets and provides rolling updates, revision history and rollback. For production application delivery, I would generally use Deployment.

---

# 51. Question 5

### Interviewer

What would you check if READY is 2 but DESIRED is 4?

### Answer

> I would first determine which two Pods are not Ready. Then I would use `kubectl describe pod`, `kubectl logs` and Kubernetes events to identify whether the issue is related to container startup, probes, resources, image pulling, configuration or node health.

---

# 52. Question 6

### Interviewer

What if all four Pods are Running but users cannot access the application?

### Answer

> I would not assume that Running means the application is reachable. I would check readiness, Service selectors, Service endpoints, ports, load balancer configuration, security groups, network policies and DNS. I would trace the request path from the load balancer to the Service and then to the Pod.

---

# 53. Question 7

### Interviewer

How would you make this highly available?

### Answer

> I would distribute replicas across multiple worker nodes and Availability Zones using topology spread constraints or appropriate anti-affinity. I would also use readiness probes, a PodDisruptionBudget, sufficient capacity and autoscaling. The goal is to avoid concentrating all replicas in one failure domain.

---

# 54. Question 8

### Interviewer

Why is `latest` not ideal for production?

### Answer

> The `latest` tag is mutable, so the image behind that tag can change. This can make deployments less reproducible and complicate rollback. I would prefer a versioned image tag or immutable image digest.

---

# 55. Question 9

### Interviewer

How would you troubleshoot ImagePullBackOff?

### Answer

> I would run `kubectl describe pod` and inspect the Events section. I would verify the image repository, tag, registry availability, authentication and IAM permissions if using ECR. I would also check network connectivity to the registry.

---

# 56. Question 10

### Interviewer

How would you troubleshoot CrashLoopBackOff?

### Answer

> I would inspect current and previous container logs, Pod events, exit codes, probes, configuration, environment variables, Secrets, ConfigMaps and resource limits. I would identify why the application exits rather than repeatedly restarting the Pod without understanding the failure.

---

# 57. Question 11

### Interviewer

How would you safely release a new application version?

### Answer

> I would use a Deployment with a rolling update strategy. I would build and scan an immutable image, deploy it gradually, monitor readiness and application metrics, and stop or roll back the deployment if health indicators deteriorate.

---

# 58. Question 12

### Interviewer

How do you reduce deployment blast radius?

### Answer

Use:

- Small batches
- Rolling deployments
- Canary releases
- Blue/green deployments where appropriate
- Automated health checks
- Feature flags
- Fast rollback
- Monitoring
- Progressive delivery

The principle is:

> Do not expose the entire production fleet to an unverified change simultaneously.

---

# 59. Question 13

### Interviewer

A deployment causes a sudden increase in errors. What do you do?

### Answer

> First I establish customer impact and stop further rollout. I compare the timeline with the deployment. If there is strong evidence that the new version caused the incident, I roll back to the last known-good revision while continuing investigation. After recovery, I perform RCA and add preventive controls.

---

# 60. Question 14

### Interviewer

Would you immediately restart all Pods during an incident?

### Answer

> No. Restarting Pods without identifying the failure mechanism can make an incident worse, especially if the problem is dependency overload, configuration, capacity or a bad deployment. I would first establish the failure pattern and choose the least risky mitigation.

---

# 61. Question 15

### Interviewer

How do you approach an AWS production incident?

### Answer

> I start with impact and scope, then establish a timeline and identify recent changes. I check the application, Kubernetes, AWS infrastructure and dependencies. I prioritize mitigation and service restoration, then identify the root cause and contributing factors. Finally, I implement corrective and preventive actions and verify that the fix actually reduces recurrence risk.

---

# 62. Senior-Level Production Scenario

## Scenario

A production EKS application normally runs 20 Pods.

Traffic suddenly increases.

CPU reaches 90%.

Pods begin restarting.

Users see 5xx errors.

### How would you respond?

First:

```text
Confirm impact
```

Then:

```text
Check application metrics
Check Pod status
Check node capacity
Check HPA
Check cluster autoscaling
Check logs
Check dependency health
```

Commands:

```bash
kubectl get pods
```

```bash
kubectl get nodes
```

```bash
kubectl top pods
```

```bash
kubectl top nodes
```

```bash
kubectl get hpa
```

```bash
kubectl describe hpa <hpa-name>
```

Then determine whether:

```text
Application saturation
```

or:

```text
Cluster capacity limitation
```

or:

```text
Dependency bottleneck
```

is responsible.

Possible mitigation:

- Increase application capacity
- Scale nodes
- Reduce non-critical load
- Protect dependencies
- Rate-limit traffic
- Roll back a recent change if relevant

---

# 63. Architect-Level Scenario

## Scenario

Your company wants to run a web application on EKS across three Availability Zones.

### Requirements

- High availability
- Safe deployments
- Fast rollback
- Centralized logging
- Secure image supply chain
- Automated scaling
- Minimal blast radius

### Architecture

```text
                    Route 53
                       |
                       v
               Load Balancer
                       |
                       v
                Kubernetes Service
                       |
                       v
                  Deployment
                       |
          +------------+------------+
          |            |            |
          v            v            v
         AZ-A         AZ-B         AZ-C
          |            |            |
       Pods         Pods         Pods
          |            |            |
          +------------+------------+
                       |
                       v
                 Application
```

Supporting architecture:

```text
CI/CD
  |
  v
Image Build
  |
  v
Security Scan
  |
  v
Amazon ECR
  |
  v
EKS
```

Observability:

```text
EKS
 |
 +-- Logs ------> CloudWatch
 |
 +-- Metrics ---> CloudWatch
 |
 +-- Alerts ----> Monitoring/Alerting
```

Security:

```text
IAM
 |
 v
EKS Workloads

Secrets Manager
 |
 v
Application Secrets

VPC
 |
 +-- Private Subnets
 +-- Security Groups
 +-- Network Controls
```

---

# 64. Interview Framework for Kubernetes Problems

When asked any Kubernetes troubleshooting question, use this sequence:

```text
1. Identify impact
2. Check resource status
3. Identify unhealthy objects
4. Describe the object
5. Check events
6. Check logs
7. Check dependencies
8. Check recent changes
9. Mitigate
10. Verify recovery
11. Perform RCA
12. Implement prevention
```

This framework demonstrates operational maturity.

---

# 65. Strong Senior Interview Answer Pattern

Instead of saying:

> "I will restart the Pod."

Say:

> "First I would establish the failure pattern and customer impact. I would inspect the workload status, events, logs, resource utilization and recent changes. If the evidence points to a bad deployment, I would stop the rollout and roll back to the last known-good version. If it is capacity-related, I would scale the appropriate layer. Once service is stable, I would perform RCA and implement preventive controls."

This demonstrates:

- Structured troubleshooting
- Risk management
- Incident response
- Root-cause thinking
- Production awareness

---

# 66. Final Task Verification

The task is successful when:

```text
ReplicaSet:
httpd-replicaset

Desired replicas:
4

Current replicas:
4

Ready replicas:
4

Container:
httpd-container

Image:
httpd:latest

Labels:
app=httpd_app
type=front-end
```

Successful ReplicaSet verification:

```text
NAME               DESIRED   CURRENT   READY   AGE
httpd-replicaset   4         4         4       23s
```

Successful Pod verification:

```text
NAME                     READY   STATUS    RESTARTS   AGE
httpd-replicaset-dw964   1/1     Running   0          38s
httpd-replicaset-fzhk2   1/1     Running   0          38s
httpd-replicaset-qdfxr   1/1     Running   0          38s
httpd-replicaset-svkgz   1/1     Running   0          38s
```

---

# 67. Final Interview Summary

If the interviewer asks:

## "Explain your ReplicaSet task."

Use this answer:

> "I created a Kubernetes ReplicaSet named `httpd-replicaset` using the `apps/v1` API and the `httpd:latest` image. I configured four replicas and used `app=httpd_app` and `type=front-end` as the identifying labels. The ReplicaSet selector matches the Pod template labels, allowing the ReplicaSet controller to manage those Pods correctly. After applying the manifest, I verified that the desired, current and ready replica counts were all four, and I confirmed that all four Pods were Running and Ready. From a production perspective, I would normally manage the workload through a Deployment, use immutable versioned images, configure resources and health probes, distribute Pods across failure domains, and integrate monitoring, security and progressive deployment controls."

---

# 68. Key Takeaways

```text
ReplicaSet
    |
    +-- Maintains desired Pod count
    |
    +-- Uses selectors
    |
    +-- Provides self-healing
    |
    +-- Reconciles desired vs actual state
    |
    +-- Does NOT directly run containers
```

Production architecture:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    v
Containers
```

Production engineering principles:

```text
Immutable artifacts
+
Health checks
+
Resource management
+
High availability
+
Observability
+
Progressive delivery
+
Fast rollback
+
Blast-radius reduction
+
Structured incident response
+
RCA and prevention
```

The most important interview message is:

> Kubernetes is a desired-state system. Controllers continuously reconcile the actual state toward the desired state. A ReplicaSet applies this principle specifically to maintaining the desired number of matching Pods.
