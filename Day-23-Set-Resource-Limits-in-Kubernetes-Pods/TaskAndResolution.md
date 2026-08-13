# Kubernetes Pod Resource Requests and Limits

## Task Resolution, Troubleshooting, AWS Mapping, RCA & DevOps Interview Handbook

---

# 1. Task Objective

Create a Kubernetes Pod with the following configuration:

| Requirement    | Required Value    |
| -------------- | ----------------- |
| Pod name       | `httpd-pod`       |
| Container name | `httpd-container` |
| Image          | `httpd:latest`    |
| CPU Request    | `100m`            |
| Memory Request | `15Mi`            |
| CPU Limit      | `100m`            |
| Memory Limit   | `20Mi`            |

The Kubernetes `kubectl` utility is already configured on the jump host.

---

# 2. Final Architecture

```text
Kubernetes Cluster
│
└── Namespace: default
    │
    └── Pod: httpd-pod
        │
        └── Container: httpd-container
            │
            ├── Image: httpd:latest
            │
            ├── Requests
            │   ├── CPU:    100m
            │   └── Memory: 15Mi
            │
            └── Limits
                ├── CPU:    100m
                └── Memory: 20Mi
```

---

# 3. Important Kubernetes Concepts

## Pod

A Pod is the smallest deployable unit in Kubernetes.

In this task:

```text
Pod
└── httpd-container
```

The Pod contains the application container.

---

## Container

The container runs the actual Apache HTTP Server application.

Configuration:

```text
Container name = httpd-container
Image          = httpd:latest
```

---

# 4. Resource Requests vs Limits

This is the most important concept in this task.

## Requests

Requests represent the resources Kubernetes should reserve/consider for scheduling the container.

This task specifies:

```text
CPU request    = 100m
Memory request = 15Mi
```

The scheduler uses resource requests when deciding whether a node has enough capacity to run the Pod.

---

## Limits

Limits define the maximum resource usage allowed for the container.

This task specifies:

```text
CPU limit    = 100m
Memory limit = 20Mi
```

Therefore:

```text
Requests
---------
CPU    = 100m
Memory = 15Mi

Limits
------
CPU    = 100m
Memory = 20Mi
```

---

# 5. CPU Units

Kubernetes CPU is commonly expressed in cores or millicores.

```text
1000m = 1 CPU core
100m  = 0.1 CPU core
50m   = 0.05 CPU core
```

Therefore:

```text
100m = 10% of one CPU core
```

The task requires:

```text
CPU request = 100m
CPU limit   = 100m
```

This means the requested and maximum CPU are the same.

---

# 6. Memory Units

The task uses:

```text
15Mi
20Mi
```

`Mi` means mebibytes.

For Kubernetes resource configuration, it is useful to distinguish:

```text
Mi = mebibyte
Gi = gibibyte
```

The requested memory is:

```text
15Mi
```

The maximum memory is:

```text
20Mi
```

---

# 7. Command History and Final Successful Configuration

The Pod was initially created with:

```bash
kubectl run httpd-pod --image=httpd:latest
```

Output:

```text
pod/httpd-pod created
```

The initially created container had the default container name:

```text
httpd-pod
```

The task, however, required:

```text
httpd-container
```

Therefore, the Pod was recreated using the required YAML specification.

---

# 8. Final YAML Manifest

The successful manifest was:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      resources:
        requests:
          memory: "15Mi"
          cpu: "100m"
        limits:
          memory: "20Mi"
          cpu: "100m"
```

---

# 9. Validate YAML Before Applying

Command:

```bash
kubectl apply --dry-run=client -f httpd-pod.yaml
```

Output:

```text
pod/httpd-pod created (dry run)
```

## Meaning

`--dry-run=client` validates the manifest locally without actually creating the resource.

This is useful before applying a configuration.

It helps catch:

* YAML structure problems
* Missing fields
* Invalid Kubernetes object definitions
* Formatting mistakes

---

# 10. Create the Pod

Command:

```bash
kubectl apply -f httpd-pod.yaml
```

Output:

```text
pod/httpd-pod created
```

This sends the Pod specification to the Kubernetes API Server.

---

# 11. Verify Pod Status

Command:

```bash
kubectl get pod httpd-pod
```

Output:

```text
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          ...
```

## Interpretation

```text
READY      = 1/1
STATUS     = Running
RESTARTS   = 0
```

This confirms:

* Pod was scheduled
* Container started
* Container is ready
* No restart has occurred

---

# 12. Verify Container Name

Command:

```bash
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].name}'
```

Output:

```text
httpd-container
```

This confirms that the required container name is correct.

---

# 13. Verify Image

The required image is:

```text
httpd:latest
```

The detailed Pod output confirmed:

```text
Image:          httpd:latest
```

This satisfies the requirement that the tag must explicitly be specified.

---

# 14. Verify Resource Configuration

Command:

```bash
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources}'
```

Output:

```text
{"limits":{"cpu":"100m","memory":"20Mi"},"requests":{"cpu":"100m","memory":"15Mi"}}
```

This exactly matches the requested configuration.

---

# 15. Detailed Pod Description

Command:

```bash
kubectl describe pod httpd-pod
```

Important output:

```text
Name:             httpd-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             worker-node/<node-ip>
Start Time:       Thu, 13 Aug 2026 04:44:32 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               <pod-ip>
```

The actual node name and IP have been anonymized.

---

# 16. Container Details

The Pod description showed:

```text
Containers:
  httpd-container:
    Container ID:   containerd://<container-id>
    Image:          httpd:latest
    Image ID:       docker.io/library/httpd@sha256:<digest>
    Port:           <none>
    Host Port:      <none>
```

This confirms:

```text
Container name = httpd-container
Image          = httpd:latest
Runtime        = containerd
```

---

# 17. Container State

The output showed:

```text
State:          Running
  Started:      Thu, 13 Aug 2026 04:44:33 +0000
Ready:          True
Restart Count:  0
```

Interpretation:

```text
Running
   ↓
Container process is running

Ready: True
   ↓
Container has passed the current readiness state

Restart Count: 0
   ↓
Container has not restarted
```

---

# 18. Resource Limits From Actual Output

The Pod description showed:

```text
Limits:
  cpu:     100m
  memory:  20Mi
Requests:
  cpu:        100m
  memory:     15Mi
```

This is the strongest validation because it confirms the resources stored in the actual Pod object.

---

# 19. QoS Class

The output showed:

```text
QoS Class:                   Burstable
```

Why?

Kubernetes QoS classes include:

```text
Guaranteed
Burstable
BestEffort
```

This Pod has:

```text
CPU request    = 100m
CPU limit      = 100m

Memory request = 15Mi
Memory limit   = 20Mi
```

Because the CPU and memory request/limit pairs are not identical for every resource, the Pod is classified as:

```text
Burstable
```

---

# 20. QoS Classes

## BestEffort

No CPU or memory requests/limits are specified.

```text
Requests = none
Limits   = none
```

These Pods have the lowest resource guarantees.

---

## Burstable

Some resource requests or limits are specified, but the Pod does not meet the strict requirements for Guaranteed QoS.

This task produces:

```text
Burstable
```

---

## Guaranteed

For a single-container Pod, to qualify as Guaranteed, CPU and memory requests and limits need to be specified and equal for every container/resource.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "20Mi"
  limits:
    cpu: "100m"
    memory: "20Mi"
```

---

# 21. Pod Scheduling

The output showed:

```text
Successfully assigned default/httpd-pod to worker-node
```

The scheduler evaluated the Pod's resource request:

```text
CPU    = 100m
Memory = 15Mi
```

The scheduler uses requests when determining whether a node can accommodate the Pod.

Conceptually:

```text
Pod
 |
 | requests:
 | CPU = 100m
 | Memory = 15Mi
 v
Scheduler
 |
 v
Find suitable node
 |
 v
Worker Node
```

---

# 22. Image Pull Lifecycle

The events showed:

```text
Pulling image "httpd:latest"
Successfully pulled image "httpd:latest"
```

Then:

```text
Created container: httpd-container
Started container httpd-container
```

The simplified lifecycle is:

```text
Pod scheduled
      ↓
Kubelet
      ↓
Container runtime
      ↓
Pull httpd:latest
      ↓
Create httpd-container
      ↓
Start container
      ↓
Container Running
```

---

# 23. Container Runtime

The output showed:

```text
Container ID:
containerd://<container-id>
```

This indicates that the node is using:

```text
containerd
```

as the container runtime.

Important distinction:

```text
kubectl
   ↓
API Server
   ↓
Pod specification
   ↓
Scheduler
   ↓
Kubelet
   ↓
containerd
   ↓
Container
```

`kubectl` does not directly create the Linux container.

The kubelet instructs the configured container runtime.

---

# 24. Why the First Pod Had the Wrong Container Name

The initial command was:

```bash
kubectl run httpd-pod --image=httpd:latest
```

The resulting container name was:

```text
httpd-pod
```

But the requirement was:

```text
httpd-container
```

There is no normal `kubectl rename container` command.

The Pod therefore had to be recreated using a manifest defining:

```yaml
containers:
  - name: httpd-container
```

This is an important Kubernetes concept:

> Many Pod specification fields cannot simply be modified in place after creation.

For immutable or restricted fields, the usual approach is:

```text
Delete old Pod
      ↓
Correct manifest
      ↓
Create new Pod
```

---

# 25. Important Difference: Pod vs Container

A common interview mistake is saying:

> "I created the container using kubectl."

More accurate:

```text
kubectl command
       ↓
Pod specification
       ↓
Kubernetes API
       ↓
Scheduler
       ↓
Kubelet
       ↓
Container runtime
       ↓
Container
```

The container exists as part of the Pod specification.

---

# 26. Why Requests Matter

Suppose a node has:

```text
Available CPU = 500m
Available Memory = 500Mi
```

A new Pod requests:

```text
CPU = 100m
Memory = 15Mi
```

The scheduler can consider the node suitable if its allocatable/requested capacity allows the Pod to fit.

Requests are therefore primarily important for:

* Scheduling
* Capacity planning
* Resource guarantees

---

# 27. Why Limits Matter

The limits specify the maximum resource usage allowed for the container.

This task specifies:

```text
CPU limit    = 100m
Memory limit = 20Mi
```

### CPU

CPU is generally throttled when the container attempts to use more than its configured CPU limit.

### Memory

Memory behaves differently.

If a container exceeds its memory limit and cannot reclaim enough memory, it can be terminated by the kernel/Kubernetes environment with an OOM-related failure.

Therefore:

```text
CPU limit exceeded
       ↓
CPU throttling

Memory limit exceeded
       ↓
Potential OOM termination
```

---

# 28. Request vs Limit — Interview Answer

### Question

What is the difference between CPU/memory requests and limits?

### Answer

**Requests** tell Kubernetes how much resource the container needs for scheduling and resource accounting.

**Limits** define the maximum amount of the resource the container is allowed to consume.

Example:

```text
Request:
CPU    = 100m
Memory = 15Mi

Limit:
CPU    = 100m
Memory = 20Mi
```

The scheduler primarily uses the request when selecting a node.

---

# 29. What Happens If Memory Usage Reaches 20Mi?

The configured memory limit is:

```text
20Mi
```

If the container attempts to exceed its memory limit and cannot reclaim memory, the container may be terminated.

Possible result:

```text
OOMKilled
```

Investigate with:

```bash
kubectl describe pod httpd-pod
```

and:

```bash
kubectl get pod httpd-pod -o jsonpath='{.status.containerStatuses[0].lastState}'
```

---

# 30. What Happens If CPU Usage Reaches 100m?

The CPU limit is:

```text
100m
```

If the container tries to use more CPU than its limit, CPU is generally throttled rather than immediately killed.

Conceptually:

```text
Application wants:
300m CPU

Limit:
100m

Result:
CPU throttling
```

This is different from memory behavior.

---

# 31. Troubleshooting — Pod Pending

Check:

```bash
kubectl get pod httpd-pod
```

Then:

```bash
kubectl describe pod httpd-pod
```

Check:

```bash
kubectl get nodes
```

And:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common causes:

* Insufficient CPU
* Insufficient memory
* Node unavailable
* Taints
* Affinity rules
* Resource constraints
* Storage constraints

---

# 32. Troubleshooting — ImagePullBackOff

Run:

```bash
kubectl describe pod httpd-pod
```

Look at Events.

Possible causes:

```text
Wrong image
Wrong tag
Registry unavailable
Authentication failure
Network problem
```

For this task verify:

```text
httpd:latest
```

---

# 33. Troubleshooting — CrashLoopBackOff

Run:

```bash
kubectl logs httpd-pod
```

Then:

```bash
kubectl logs httpd-pod --previous
```

Then:

```bash
kubectl describe pod httpd-pod
```

Investigate:

* Application startup
* Configuration
* Permissions
* Environment variables
* Dependency failures
* Probes
* Resource limits

---

# 34. Troubleshooting — OOMKilled

Check:

```bash
kubectl describe pod httpd-pod
```

Look for:

```text
OOMKilled
```

Also:

```bash
kubectl get pod httpd-pod -o jsonpath='{.status.containerStatuses[0].lastState}'
```

Possible root causes:

* Memory limit too low
* Memory leak
* Unexpected workload
* Large request processing
* Incorrect application configuration

Resolution should not simply be:

```text
Increase memory limit
```

First determine why the application consumed more memory.

---

# 35. Troubleshooting — CPU Throttling

Symptoms may include:

* Increased application latency
* Slow request processing
* CPU usage appearing capped
* Poor throughput

Check resource configuration:

```bash
kubectl get pod httpd-pod -o yaml
```

Check actual metrics if metrics-server/observability is available:

```bash
kubectl top pod httpd-pod
```

Then compare:

```text
Requested CPU
Actual CPU
CPU Limit
```

---

# 36. Important Commands

## Pod

```bash
kubectl get pod httpd-pod
```

## Detailed Pod

```bash
kubectl describe pod httpd-pod
```

## YAML

```bash
kubectl get pod httpd-pod -o yaml
```

## Container name

```bash
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].name}'
```

## Image

```bash
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].image}'
```

## Resources

```bash
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources}'
```

## Logs

```bash
kubectl logs httpd-pod
```

## Previous container logs

```bash
kubectl logs httpd-pod --previous
```

## Node information

```bash
kubectl get pod httpd-pod -o wide
```

## Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Metrics

```bash
kubectl top pod httpd-pod
```

---

# 37. L1 Interview — Junior

## Q1. What is a resource request?

A resource request specifies the amount of CPU or memory Kubernetes should consider when scheduling a Pod.

Example:

```yaml
requests:
  cpu: "100m"
  memory: "15Mi"
```

---

## Q2. What is a resource limit?

A resource limit defines the maximum resource usage allowed for a container.

Example:

```yaml
limits:
  cpu: "100m"
  memory: "20Mi"
```

---

## Q3. What is 100m CPU?

`100m` means 100 millicores:

```text
1000m = 1 CPU
100m  = 0.1 CPU
```

---

## Q4. What does 15Mi mean?

It represents 15 mebibytes of memory.

---

## Q5. What happens when CPU exceeds the limit?

CPU can be throttled.

---

## Q6. What can happen when memory exceeds the limit?

The container can be terminated due to an out-of-memory condition.

---

# 38. L2 Interview — Intermediate

## Q7. Does Kubernetes schedule based on limits or requests?

The scheduler primarily uses resource **requests** when determining whether a Pod can fit on a node.

Example:

```text
Node allocatable:
CPU = 2 cores

Existing requests:
CPU = 1.8 cores

New Pod request:
CPU = 500m
```

The scheduler should not place the Pod there if doing so would exceed allocatable requested capacity.

---

## Q8. Why should requests be realistic?

If requests are too high:

```text
Cluster utilization ↓
Scheduling failures ↑
Cost ↑
```

If requests are too low:

```text
Node contention ↑
Performance unpredictability ↑
Eviction risk ↑
```

Therefore requests should be based on observed workload behavior.

---

## Q9. Why should limits be configured carefully?

Very low limits can cause:

* CPU throttling
* OOMKills
* Latency
* Application instability

Very high limits can reduce effective cluster resource governance.

---

## Q10. What QoS class does this Pod have?

This Pod has:

```text
QoS Class: Burstable
```

because its requests and limits are not identical for all specified resources.

---

# 39. L3 Interview — Senior

## Q11. A Pod is Pending even though the node has free memory. Why?

"Free memory" as observed from the OS does not necessarily mean sufficient Kubernetes **allocatable/request capacity** exists.

Check:

```bash
kubectl describe node <node>
```

Look at:

```text
Allocatable
Requests
Limits
```

Also check:

```bash
kubectl describe pod <pod>
```

and events.

Other reasons include:

* Taints
* Node affinity
* Pod affinity
* Topology constraints
* Resource type mismatch

---

## Q12. Application is slow after adding CPU limits. What could happen?

The CPU limit may be causing throttling.

For example:

```text
Application demand = 500m
CPU limit          = 100m
```

The application can experience throttling and increased latency.

Investigate:

```bash
kubectl top pod <pod>
```

and application metrics.

---

## Q13. Application was OOMKilled after traffic increased. What do you investigate?

Do not immediately increase the limit.

Investigate:

1. Memory usage trend
2. Application memory behavior
3. Traffic increase
4. Request size
5. Memory leak
6. JVM/runtime configuration if applicable
7. Container memory limit
8. Node memory pressure

Then determine the correct resource configuration.

---

# 40. L4 Interview — Architect

## Q14. How would you establish resource requests and limits for production workloads?

Use a data-driven process:

```text
Deploy
 ↓
Observe
 ↓
Measure CPU/Memory
 ↓
Identify P50/P95/P99
 ↓
Set Requests
 ↓
Set Limits
 ↓
Load Test
 ↓
Monitor
 ↓
Tune
```

Use:

* Historical metrics
* Load testing
* Production traffic patterns
* Capacity planning
* Application profiling

Avoid blindly copying resource values between applications.

---

## Q15. What happens if every application sets extremely high limits?

Limits are not necessarily reserved capacity.

However, excessively high configurations can make capacity planning and scheduling less predictable and can allow workloads to compete for node resources.

Use:

* Requests
* Limits
* ResourceQuota
* LimitRange
* Autoscaling
* Capacity planning

together.

---

## Q16. How would you prevent one team from consuming the entire cluster?

Use Namespace-level controls:

```text
Namespace
   |
   +-- ResourceQuota
   |
   +-- LimitRange
   |
   +-- RBAC
```

Example:

```text
Development namespace
---------------------
Maximum Pods     = 50
CPU requests     = 10 cores
Memory requests  = 20Gi
```

This provides resource governance.

---

# 41. ResourceQuota vs LimitRange

## ResourceQuota

Controls aggregate resource usage within a Namespace.

Example:

```text
Namespace total CPU requests <= 10 cores
Namespace total memory requests <= 20Gi
```

---

## LimitRange

Controls defaults and allowed resource ranges for individual Pods/containers.

Conceptually:

```text
Minimum memory = 64Mi
Maximum memory = 2Gi
Default memory = 256Mi
```

Use them together for strong resource governance.

---

# 42. AWS EKS Mapping

The Kubernetes task maps to Amazon EKS.

```text
Kubernetes
     ↓
Amazon EKS
```

Architecture:

```text
AWS VPC
│
├── Availability Zone A
│   └── EKS Worker Node
│       └── Pod
│
├── Availability Zone B
│   └── EKS Worker Node
│       └── Pod
│
└── Availability Zone C
    └── EKS Worker Node
        └── Pod
```

---

# 43. Kubernetes Resources vs AWS Capacity

In EKS, Kubernetes resource requests eventually map to capacity available on worker nodes.

For example:

```text
Pod request:
CPU = 100m
Memory = 15Mi
```

The EKS worker node must have sufficient allocatable resources.

Worker capacity may come from:

* EC2 instances
* EKS managed node groups
* Karpenter-provisioned nodes
* Other supported compute models

---

# 44. AWS Scenario — Pods Are Pending

## Scenario

An EKS application suddenly has many Pending Pods.

### Investigation

```bash
kubectl get pods
kubectl describe pod <pending-pod>
kubectl get nodes
kubectl describe nodes
```

If the reason is:

```text
Insufficient cpu
```

then investigate:

```text
Pod requests
Node allocatable CPU
Current requested CPU
Autoscaling
```

Possible solutions:

* Add nodes
* Use larger nodes
* Adjust requests based on evidence
* Configure Cluster Autoscaler/Karpenter
* Optimize application resource usage

---

# 45. AWS Scenario — EKS Pod OOMKilled

### Symptoms

```text
Pod restarts
OOMKilled
```

Investigate:

```bash
kubectl describe pod <pod>
kubectl top pod <pod>
kubectl logs <pod> --previous
```

AWS-side investigation may include:

* CloudWatch metrics
* Node memory pressure
* EC2 instance size
* EKS node group capacity
* Application metrics

Root cause could be:

```text
Application memory growth
+
Low container memory limit
```

Do not automatically assume the EC2 instance needs to be larger.

The container limit may be the actual constraint.

---

# 46. AWS Scenario — EKS CPU Throttling

Suppose:

```text
CPU request = 100m
CPU limit   = 100m
```

The application needs:

```text
250m
```

The container may experience CPU throttling.

Symptoms:

```text
Latency ↑
Throughput ↓
CPU throttling ↑
```

Possible corrective actions:

* Reassess CPU request
* Reassess CPU limit
* Scale horizontally
* Optimize application
* Configure HPA
* Increase node capacity if required

---

# 47. AWS Scenario — Cluster Cost Is Increasing

Suppose the cluster has:

```text
Pods = 100
CPU requests = very high
Memory requests = very high
```

The scheduler may require more worker nodes than actually necessary.

Investigate:

```text
Pod requests
Node utilization
Node allocatable capacity
Actual CPU usage
Actual memory usage
```

Possible solution:

```text
Right-size requests
+
Autoscaling
+
Appropriate node instance types
```

This can improve both:

```text
Performance
Cost
```

---

# 48. AWS Scenario — Node Memory Pressure

If nodes experience memory pressure:

```text
Node memory pressure
       ↓
Eviction decisions
       ↓
Lower-priority workloads may be evicted
```

Investigate:

```bash
kubectl describe node <node>
kubectl get pods -o wide
kubectl top nodes
kubectl top pods
```

Then determine:

* Which workloads consume memory
* Whether requests are accurate
* Whether limits are too high
* Whether nodes are correctly sized
* Whether autoscaling is required

---

# 49. Real AWS Incident RCA #1 — S3 US-EAST-1, February 2017

## Incident

On February 28, 2017, Amazon S3 experienced a major service disruption in US-EAST-1.

AWS's official post-event summary explains that an authorized operator executed an established operational command while debugging a billing-system issue.

An incorrect input caused more servers than intended to be removed.

Those servers supported multiple S3 subsystems.

One affected subsystem was the S3 index subsystem responsible for metadata and object location information.

### Simplified chain

```text
Operational command
       ↓
Incorrect input
       ↓
Too many servers removed
       ↓
Multiple S3 subsystems affected
       ↓
Index subsystem degraded
       ↓
S3 API impact
       ↓
Dependent AWS services affected
```

AWS reported that the index subsystem began recovering before the placement subsystem, with full S3 recovery later in the event.

### Root Cause

An operational action removed a larger set of servers than intended.

### Contributing Factor

The affected subsystems had not required a complete restart at that scale for many years, while the service had grown significantly.

### DevOps Lessons

1. Production automation must have guardrails.
2. Dangerous commands should require validation.
3. Blast radius should be limited.
4. Operational playbooks need safety checks.
5. Large-scale recovery procedures must be regularly tested.
6. Capacity growth can invalidate old recovery assumptions.

### Kubernetes Connection

The same principles apply to Kubernetes:

```text
kubectl delete
kubectl rollout
kubectl drain
kubectl scale
```

can have significant blast radius.

Production environments should use:

* RBAC
* Approval workflows
* GitOps
* Change control
* Admission policies
* Namespace isolation
* PodDisruptionBudgets
* Automated validation

Source: AWS official S3 service disruption summary.

---

# 50. Real AWS Incident RCA #2 — US-EAST-1, December 2021

## Incident

AWS experienced a major service event in Northern Virginia on December 7, 2021.

AWS reported that an automated scaling activity triggered unexpected behavior in clients communicating through internal networking infrastructure.

The resulting connection surge overwhelmed networking devices between an internal AWS network and the main AWS network.

### Simplified chain

```text
Automated scaling
       ↓
Unexpected client behavior
       ↓
Connection surge
       ↓
Network device congestion
       ↓
Latency + errors
       ↓
Retries increased traffic
       ↓
Persistent congestion
       ↓
Multiple AWS service impacts
```

### Important Control Plane vs Data Plane Lesson

AWS reported that existing EC2 instances could continue running while EC2 APIs for launching and describing resources experienced elevated errors and latency.

Similarly, existing container workloads could continue operating, but container-management operations could be affected.

This is extremely important for Kubernetes engineers.

```text
Existing workload
        |
        v
May continue running

Management API
        |
        v
May be degraded
```

### Kubernetes/EKS Lesson

A running EKS workload may continue to operate while operations such as:

```text
Creating new resources
Replacing failed capacity
Scaling
Starting new workloads
```

can be affected by control-plane/dependency failures.

### Architecture lesson

Design systems for:

* Failure of management dependencies
* Multi-AZ operation
* Graceful degradation
* Retries with backoff
* Queueing
* Capacity headroom
* Dependency isolation

Source: AWS official December 2021 US-EAST-1 service event summary.

---

# 51. Real AWS Incident RCA #3 — Kinesis Data Streams, July 2024

## Incident

On July 30, 2024, AWS reported an event involving a Kinesis Data Streams cell in US-EAST-1.

The incident affected services including:

* CloudWatch Logs
* Amazon Data Firehose
* S3 event delivery
* ECS
* Lambda
* Redshift
* Glue

### Root Cause

AWS explained that one internal Kinesis cell had an unusual workload profile containing a very large number of low-throughput shards.

A routine deployment caused hosts to be taken out of service and then brought back.

The cell-management system did not distribute this workload effectively.

A small number of hosts received a disproportionately large number of shards.

This resulted in resource contention and degradation.

### Simplified chain

```text
Routine deployment
       ↓
Hosts removed/reintroduced
       ↓
Work redistribution
       ↓
Unusual workload profile
       ↓
Poor workload distribution
       ↓
Resource contention
       ↓
Kinesis degradation
       ↓
Dependent services impacted
```

### DevOps Lesson

A service can fail because of a workload shape that was not represented in normal testing.

Therefore:

* Test unusual workload patterns.
* Monitor resource saturation.
* Use cell/AZ fault isolation.
* Avoid correlated dependencies.
* Load test deployments.
* Have rollback and mitigation procedures.

### Kubernetes Connection

This resembles Kubernetes scheduling/resource problems.

A workload may appear normal under average traffic but behave differently under:

```text
High concurrency
Large number of small requests
Large number of small Pods
High shard/task count
Uneven workload distribution
```

Therefore capacity testing should include realistic workload distributions.

Source: AWS official Kinesis Data Streams service event summary.

---

# 52. RCA Methodology for DevOps

Use this structure during incidents.

## 1. Incident

What happened?

Example:

```text
HTTP 503 errors increased.
```

## 2. Impact

Who was affected?

```text
30% of requests failed.
```

## 3. Detection

How was it detected?

```text
CloudWatch alarm
Application monitoring
Customer ticket
```

## 4. Timeline

Document:

```text
T0  Incident begins
T1  Alert triggered
T2  Investigation begins
T3  Root cause identified
T4  Mitigation applied
T5  Service recovered
```

## 5. Root Cause

Identify the technical failure.

## 6. Contributing Factors

Identify what increased the severity.

## 7. Mitigation

What restored service?

## 8. Corrective Actions

What fixes the technical issue?

## 9. Preventive Actions

What reduces the probability or blast radius?

---

# 53. Full DevOps Mock Interview

## Question 1

Create the required Pod.

### Answer

```bash
kubectl apply -f httpd-pod.yaml
```

The manifest specifies:

```text
Pod:
httpd-pod

Container:
httpd-container

Image:
httpd:latest

Requests:
CPU    = 100m
Memory = 15Mi

Limits:
CPU    = 100m
Memory = 20Mi
```

---

## Question 2

How do you verify the Pod?

### Answer

```bash
kubectl get pod httpd-pod
```

Expected:

```text
httpd-pod   1/1   Running   0
```

---

## Question 3

How do you verify the resource configuration?

### Answer

```bash
kubectl describe pod httpd-pod
```

Look for:

```text
Limits:
  cpu:     100m
  memory:  20Mi

Requests:
  cpu:     100m
  memory:  15Mi
```

Or:

```bash
kubectl get pod httpd-pod -o jsonpath='{.spec.containers[0].resources}'
```

---

## Question 4

What is the difference between request and limit?

### Answer

Request influences scheduling and resource accounting.

Limit establishes the maximum allowed resource consumption for the container.

---

## Question 5

Why is the Pod Burstable?

### Answer

The Pod does not meet the requirements for Guaranteed QoS because its CPU and memory request/limit pairs are not both identical.

Here:

```text
CPU:
request = 100m
limit   = 100m

Memory:
request = 15Mi
limit   = 20Mi
```

Memory request and limit differ, so the Pod is Burstable.

---

## Question 6

What happens if the container uses more than 100m CPU?

### Answer

The CPU limit can cause CPU throttling.

The application may experience:

* Increased latency
* Lower throughput
* Slower processing

---

## Question 7

What happens if the container exceeds 20Mi memory?

### Answer

If it cannot reclaim enough memory, the container may be terminated with an OOM-related failure.

The Pod can subsequently restart the container depending on its restart behavior.

---

## Question 8

Why not set every application's request and limit extremely high?

### Answer

Because resource requests affect scheduling and capacity planning.

Overstated requests can cause:

```text
Low cluster utilization
More nodes required
Higher infrastructure cost
Pending Pods
```

Requests should be based on observed workload behavior.

---

## Question 9

How would you determine the correct resource values?

### Answer

I would:

1. Start with baseline estimates.
2. Deploy the application.
3. Measure CPU and memory.
4. Test normal and peak traffic.
5. Review P95/P99 behavior.
6. Check throttling and OOM events.
7. Tune requests and limits.
8. Repeat under realistic load.

---

## Question 10

A Pod is Pending because of insufficient CPU. What do you check?

### Answer

```bash
kubectl describe pod <pod>
kubectl get nodes
kubectl describe node <node>
```

Check:

```text
CPU requests
Node allocatable CPU
Current Pod requests
Node count
Autoscaling
```

---

## Question 11

A Pod is OOMKilled. What is your approach?

### Answer

First verify:

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod <pod>
```

Then determine whether the root cause is:

```text
Application memory leak
Traffic increase
Incorrect memory request
Low memory limit
Node memory pressure
```

Only then change the resource configuration.

---

## Question 12

How would you handle this in EKS?

### Answer

I would use:

```text
Amazon EKS
    |
    +-- Multiple AZs
    |
    +-- EKS Worker Nodes
    |
    +-- Kubernetes Resource Requests/Limits
    |
    +-- HPA
    |
    +-- Node Autoscaling/Karpenter
    |
    +-- CloudWatch/Observability
```

I would ensure that:

* Pod requests reflect actual workload needs.
* Node capacity is sufficient.
* Autoscaling is configured.
* Applications are spread across AZs.
* ResourceQuota is used where appropriate.

---

# 54. Architect-Level Scenario

## Scenario

A production EKS application has:

```text
CPU request = 100m
CPU limit   = 100m
```

During peak traffic:

```text
Latency increases
CPU throttling increases
```

### Analysis

The container may be CPU-constrained.

The first question should not be:

> "Why is the EC2 instance too small?"

Instead ask:

```text
What is actual application CPU demand?
What is the container CPU limit?
What is the CPU request?
How many replicas exist?
Is HPA configured?
Is the application horizontally scalable?
```

Potential solution:

```text
Measure
 ↓
Right-size request/limit
 ↓
Configure HPA
 ↓
Add replicas
 ↓
Ensure node capacity
```

---

# 55. Architect-Level Resource Strategy

A production platform can use:

```text
Namespace
   |
   +-- ResourceQuota
   |
   +-- LimitRange
   |
   +-- Deployments
   |
   +-- Requests/Limits
   |
   +-- HPA
   |
   +-- Node Autoscaling
   |
   +-- Monitoring
```

This creates multiple layers of resource governance.

---

# 56. Resource Governance Model

```text
Application
     |
     v
Container Requests/Limits
     |
     v
Pod
     |
     v
Namespace ResourceQuota
     |
     v
Node Allocatable Capacity
     |
     v
Cluster Capacity
     |
     v
AWS EC2 Capacity
```

Each layer has a different responsibility.

---

# 57. Important Production Best Practices

## 1. Always define requests for production workloads

Without requests, scheduling and capacity planning become less predictable.

---

## 2. Do not blindly use very low limits

Low limits can cause:

```text
CPU throttling
OOMKills
Latency
```

---

## 3. Do not blindly use extremely high requests

This can lead to:

```text
Pending Pods
Underutilized nodes
Higher AWS cost
```

---

## 4. Measure before tuning

Use:

```bash
kubectl top pod
kubectl top nodes
```

along with application and infrastructure monitoring.

---

## 5. Use autoscaling

For suitable workloads:

```text
HPA
+
Node Autoscaling
```

can allow capacity to respond to demand.

---

## 6. Use ResourceQuota

Protect a cluster from one namespace consuming excessive resources.

---

## 7. Use LimitRange

Define defaults and boundaries for containers in a Namespace.

---

# 58. Important Commands Cheat Sheet

```bash
# Create/Apply
kubectl apply -f httpd-pod.yaml

# Validate without creating
kubectl apply --dry-run=client -f httpd-pod.yaml

# Pod status
kubectl get pod httpd-pod

# Detailed information
kubectl describe pod httpd-pod

# Full YAML
kubectl get pod httpd-pod -o yaml

# Container name
kubectl get pod httpd-pod \
  -o jsonpath='{.spec.containers[0].name}'

# Image
kubectl get pod httpd-pod \
  -o jsonpath='{.spec.containers[0].image}'

# Resources
kubectl get pod httpd-pod \
  -o jsonpath='{.spec.containers[0].resources}'

# Logs
kubectl logs httpd-pod

# Previous logs
kubectl logs httpd-pod --previous

# Pod/node information
kubectl get pod httpd-pod -o wide

# Events
kubectl get events --sort-by=.metadata.creationTimestamp

# Resource metrics
kubectl top pod httpd-pod
kubectl top nodes
```

---

# 59. Final Validation

## Pod

```text
Name:
httpd-pod
```

## Container

```text
Name:
httpd-container
```

## Image

```text
httpd:latest
```

## Requests

```text
CPU:
100m

Memory:
15Mi
```

## Limits

```text
CPU:
100m

Memory:
20Mi
```

## Status

```text
Running
```

## Ready

```text
True
```

## Restarts

```text
0
```

## QoS

```text
Burstable
```

---

# 60. Final Task Configuration

```text
Pod: httpd-pod
│
├── Namespace: default
│
└── Container: httpd-container
    │
    ├── Image: httpd:latest
    │
    ├── Requests
    │   ├── CPU:    100m
    │   └── Memory: 15Mi
    │
    └── Limits
        ├── CPU:    100m
        └── Memory: 20Mi
```

---

# 61. Final Interview Mental Model

Remember:

```text
REQUEST
   ↓
Used primarily for scheduling
   ↓
"How much does this workload need?"

LIMIT
   ↓
Maximum allowed usage
   ↓
"How much can this container consume?"

CPU LIMIT
   ↓
Possible throttling

MEMORY LIMIT
   ↓
Possible OOM termination

REQUESTS + LIMITS
   ↓
QoS classification
   ↓
Resource governance
```

---

# 62. Most Important Interview Points

1. **Requests are primarily used by the scheduler.**
2. **Limits define the maximum allowed container resource usage.**
3. **CPU exceeding its limit generally results in throttling.**
4. **Memory exceeding its limit can result in OOM termination.**
5. **Resource requests should be based on actual workload requirements.**
6. **Resource limits should not be arbitrarily low.**
7. **ResourceQuota controls aggregate Namespace consumption.**
8. **LimitRange can establish defaults and boundaries.**
9. **HPA scales Pods; node autoscaling adds infrastructure capacity.**
10. **A Pod with mismatched resource request/limit pairs can be Burstable.**
11. **`kubectl describe pod` is one of the first commands to use during troubleshooting.**
12. **`kubectl logs --previous` is particularly useful after container crashes.**
13. **In EKS, Kubernetes resource management ultimately depends on available worker-node capacity.**
14. **Right-sizing resources improves both reliability and cloud cost efficiency.**
15. **Never increase resource limits blindly; identify the actual bottleneck first.**

---

# 63. One-Minute Interview Answer

If an interviewer asks:

> "Explain what you configured in this task."

A strong answer is:

> "I created a Kubernetes Pod named `httpd-pod` with a container named `httpd-container` using the explicitly tagged `httpd:latest` image. I configured a CPU request and limit of 100 millicores, a memory request of 15Mi, and a memory limit of 20Mi. The Pod reached Running and Ready state with zero restarts. The resource requests are used by Kubernetes during scheduling and capacity planning, while the limits constrain the container's maximum resource consumption. Because the memory request and limit differ, the Pod has Burstable QoS. In production I would establish these values from workload metrics and load testing rather than choosing them arbitrarily, and in EKS I would combine them with HPA, node autoscaling, ResourceQuota, monitoring, and multi-AZ capacity planning."

---

# 64. Final Outcome

The final Pod configuration is correct:

```text
Pod                 = httpd-pod
Container           = httpd-container
Image               = httpd:latest
CPU Request         = 100m
Memory Request      = 15Mi
CPU Limit           = 100m
Memory Limit        = 20Mi
Status              = Running
Ready               = True
Restart Count       = 0
QoS Class            = Burstable
```

The successful Kubernetes lifecycle was:

```text
YAML Manifest
     ↓
API Server
     ↓
Pod Object
     ↓
Scheduler
     ↓
Worker Node
     ↓
Kubelet
     ↓
containerd
     ↓
Pull httpd:latest
     ↓
Create httpd-container
     ↓
Start Container
     ↓
Running / Ready
```

The core DevOps lesson is:

```text
Resource Requests
        +
Resource Limits
        +
Monitoring
        +
Autoscaling
        +
Capacity Planning
        =
Predictable Kubernetes Workloads
```

