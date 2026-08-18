# Kubernetes Rolling Update — TaskAndResolution.md

# 1. Task

## Requirement

An existing Kubernetes Deployment named `nginx-deployment` was running an older NGINX image.

The requirement was to perform a **Rolling Update** using:

- Deployment: `nginx-deployment`
- Container: `nginx-container`
- New Image: `nginx:1.18`
- Replicas: `3`
- All Pods must be operational after the update.

---

# 2. Kubernetes Rolling Update — Concept

A Rolling Update replaces old Pods gradually with new Pods instead of stopping all Pods at once.

```text
Before:

Deployment
   |
   +-- ReplicaSet
        |
        +-- Pod 1 -> nginx:1.16
        +-- Pod 2 -> nginx:1.16
        +-- Pod 3 -> nginx:1.16


During Rolling Update:

Deployment
   |
   +-- Old ReplicaSet
   |      |
   |      +-- nginx:1.16
   |
   +-- New ReplicaSet
          |
          +-- nginx:1.18


After:

Deployment
   |
   +-- New ReplicaSet
        |
        +-- Pod 1 -> nginx:1.18
        +-- Pod 2 -> nginx:1.18
        +-- Pod 3 -> nginx:1.18
```

The important Kubernetes relationship is:

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

---

# 3. Verify Existing Deployment

## Command

```bash
kubectl get deployment nginx-deployment
```

## Output

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           ...
```

## Explanation

- `READY 3/3` = all 3 desired Pods are ready.
- `UP-TO-DATE 3` = all 3 replicas match the current Deployment specification.
- `AVAILABLE 3` = all 3 replicas are available.

The Deployment was healthy before the update.

---

# 4. Verify Existing Pods

## Command

```bash
kubectl get pods
```

## Output

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-<old-rs>-<pod1>   1/1     Running   0          ...
nginx-deployment-<old-rs>-<pod2>   1/1     Running   0          ...
nginx-deployment-<old-rs>-<pod3>   1/1     Running   0          ...
```

The actual Pod names are environment-specific and have been anonymized.

## Explanation

All existing Pods were:

```text
READY   = 1/1
STATUS  = Running
```

Therefore the application was healthy before the deployment.

---

# 5. Check Current Image

## Command

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Output

```text
nginx:1.16
```

Therefore:

```text
Current Image  = nginx:1.16
Required Image = nginx:1.18
```

---

# 6. Check Container Name

## Command

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'
```

## Output

```text
nginx-container
```

Therefore the container name required for the update is:

```text
nginx-container
```

---

# 7. Perform Rolling Update

## Command

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18
```

## Output

```text
deployment.apps/nginx-deployment image updated
```

## Explanation

`kubectl set image` changes the container image in the Deployment's Pod template.

Kubernetes detects the Pod-template change and creates a new ReplicaSet.

It then gradually replaces Pods managed by the old ReplicaSet with Pods managed by the new ReplicaSet.

---

# 8. Monitor Rolling Update

## Command

```bash
kubectl rollout status deployment/nginx-deployment
```

During the rollout, Kubernetes can display progress such as:

```text
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
```

Then:

```text
Waiting for deployment "nginx-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
```

Then:

```text
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
```

Finally:

```text
deployment "nginx-deployment" successfully rolled out
```

## Explanation

The messages show Kubernetes gradually replacing the old Pods.

Conceptually:

```text
Old: 3    New: 0
       |
       v
Old: 2    New: 1
       |
       v
Old: 1    New: 2
       |
       v
Old: 0    New: 3
```

The exact sequence depends on:

- `maxSurge`
- `maxUnavailable`
- Replica count
- Pod readiness
- Cluster capacity

---

# 9. Verify Pods After Rolling Update

## Command

```bash
kubectl get pods
```

## Final Output

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-<new-rs>-<pod1>   1/1     Running   0          ...
nginx-deployment-<new-rs>-<pod2>   1/1     Running   0          ...
nginx-deployment-<new-rs>-<pod3>   1/1     Running   0          ...
```

## Explanation

All three Pods are:

```text
READY   = 1/1
STATUS  = Running
```

Therefore:

```text
3/3 Pods are operational.
```

---

# 10. Verify Deployment After Update

## Command

```bash
kubectl get deployment nginx-deployment
```

## Output

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           ...
```

## Explanation

```text
READY       = 3/3
UP-TO-DATE  = 3
AVAILABLE   = 3
```

This confirms that all three desired replicas are ready and available.

---

# 11. Verify Final Image

## Command

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Output

```text
nginx:1.18
```

This confirms that the Deployment is configured with the required image.

---

# 12. Final Task Result

```text
Deployment:       nginx-deployment
Container:        nginx-container
Old Image:        nginx:1.16
New Image:        nginx:1.18
Desired Replicas: 3
Ready:            3/3
Available:        3
Pod Status:       Running
Rollout:          Successfully rolled out
```

## Final Status

```text
TASK COMPLETED SUCCESSFULLY
```

---

# 13. Complete Command Sequence

```bash
# 1. Check Deployment
kubectl get deployment nginx-deployment

# 2. Check Pods
kubectl get pods

# 3. Check current image
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

# 4. Check container name
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'

# 5. Perform Rolling Update
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18

# 6. Monitor rollout
kubectl rollout status deployment/nginx-deployment

# 7. Verify Pods
kubectl get pods

# 8. Verify Deployment
kubectl get deployment nginx-deployment

# 9. Verify final image
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# 14. Why Rolling Update Is Used

Without Rolling Update:

```text
Stop old application
        |
        v
Start new application
```

This can create downtime.

With Rolling Update:

```text
Old Old Old
    |
    v
Old Old New
    |
    v
Old New New
    |
    v
New New New
```

This allows Kubernetes to maintain application capacity while the new version is deployed.

---

# 15. Deployment vs ReplicaSet vs Pod

## Deployment

Responsible for:

- Application deployment
- Rolling Updates
- Rollbacks
- Revision history
- Replica management

## ReplicaSet

Responsible for:

- Maintaining the desired number of Pods.

## Pod

The smallest deployable Kubernetes unit.

A Pod contains one or more containers.

Relationship:

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

---

# 16. Why Pod Names Change

When the Deployment Pod template changes, Kubernetes creates a new ReplicaSet.

The new ReplicaSet has a different Pod-template hash.

Therefore new Pods receive new names.

Example:

```text
Old ReplicaSet
    |
    +-- application-oldhash-pod1
    +-- application-oldhash-pod2
    +-- application-oldhash-pod3
```

After update:

```text
New ReplicaSet
    |
    +-- application-newhash-pod1
    +-- application-newhash-pod2
    +-- application-newhash-pod3
```

This is normal Kubernetes behavior.

---

# 17. RollingUpdate Configuration

A Deployment can use:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```

## maxUnavailable

Defines how many desired Pods can be unavailable during the update.

## maxSurge

Defines how many additional Pods can temporarily exist above the desired replica count.

Example:

```text
Desired replicas = 3
```

During the update Kubernetes may temporarily have more than 3 Pods depending on `maxSurge`.

---

# 18. Readiness Probe

A Pod being `Running` does not necessarily mean the application is ready to receive traffic.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 5
```

Concept:

```text
Pod Created
    |
    v
Container Started
    |
    v
Application Initialization
    |
    v
Readiness Check
    |
    +---- Failure ---> Not Ready
    |
    +---- Success ---> Ready
```

Readiness probes are very important during Rolling Updates.

---

# 19. Readiness vs Liveness

## Readiness

Question:

```text
Is the application ready to receive traffic?
```

Example:

```text
READY = 0/1
STATUS = Running
```

The container is running but is not ready.

## Liveness

Question:

```text
Is the application still alive/healthy?
```

If liveness repeatedly fails, Kubernetes may restart the container.

Easy memory:

```text
Readiness = Ready for traffic?
Liveness  = Still alive?
```

---

# 20. Rollout History

Check Deployment revisions:

```bash
kubectl rollout history deployment/nginx-deployment
```

This is useful when investigating previous versions.

---

# 21. Rollback

If the new application version causes problems:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Then:

```bash
kubectl rollout status deployment/nginx-deployment
```

And:

```bash
kubectl get pods
```

Rollback restores the previous Deployment revision.

---

# 22. Rollback to Specific Revision

Check history:

```bash
kubectl rollout history deployment/nginx-deployment
```

Rollback:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

Verify:

```bash
kubectl rollout status deployment/nginx-deployment
```

---

# 23. Troubleshooting a Stuck Rollout

If:

```bash
kubectl rollout status deployment/nginx-deployment
```

does not complete, use:

```bash
kubectl get pods
```

```bash
kubectl describe deployment nginx-deployment
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common causes:

```text
ImagePullBackOff
ErrImagePull
CrashLoopBackOff
Pending
Readiness probe failure
Insufficient CPU
Insufficient memory
Configuration error
Application startup failure
Network problem
Dependency failure
```

---

# 24. Scenario-Based Interview Questions — L1

## Q1. What is a Rolling Update?

A Rolling Update gradually replaces old application Pods with new Pods while maintaining application availability.

## Q2. Which Kubernetes object performs Rolling Updates?

A Deployment.

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

## Q3. Which command updates the image?

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18
```

## Q4. How do you monitor a rollout?

```bash
kubectl rollout status deployment/nginx-deployment
```

## Q5. How do you verify the image?

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Expected:

```text
nginx:1.18
```

---

# 25. Scenario-Based Interview Questions — L2

## Q6. What is a Deployment?

A Deployment is a Kubernetes workload controller used to declaratively manage application Pods.

It provides:

- Replica management
- Rolling Updates
- Rollbacks
- Revision history
- Self-healing through ReplicaSets

## Q7. What is a ReplicaSet?

A ReplicaSet ensures that the desired number of Pods exists.

Example:

```text
replicas = 3
```

The ReplicaSet attempts to maintain three Pods.

## Q8. Why did the Pod names change?

The Deployment created a new ReplicaSet with a different Pod-template hash.

The new ReplicaSet created new Pods.

## Q9. What is maxUnavailable?

The maximum number or percentage of desired Pods that can be unavailable during the update.

## Q10. What is maxSurge?

The maximum number or percentage of additional Pods that can temporarily exist above the desired replica count.

---

# 26. Scenario-Based Interview Questions — L3

## Q11. The rollout is stuck. How do you troubleshoot?

I would execute:

```bash
kubectl rollout status deployment/nginx-deployment
kubectl get pods
kubectl describe deployment nginx-deployment
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Then identify whether the issue is:

```text
Image
Application
Readiness
Scheduling
Resources
Networking
Configuration
Dependencies
```

## Q12. New Pods are Running but users receive errors. What do you check?

I would check:

```text
Pod readiness
Service endpoints
Ingress
Load Balancer
Application logs
Application health
Network policies
Backend dependencies
DNS
```

Commands:

```bash
kubectl get pods
kubectl get svc
kubectl get endpoints
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## Q13. Why can a Running Pod still be unable to receive traffic?

Because:

```text
Running != Ready
```

Example:

```text
READY   STATUS
0/1     Running
```

The container is running, but the application has not passed its readiness condition.

---

# 27. Scenario-Based Interview Questions — L4

## Q14. How would you design a production Rolling Update?

I would use:

```text
Deployment
Multiple replicas
RollingUpdate
Readiness probe
Startup probe where required
Appropriate maxUnavailable
Appropriate maxSurge
PodDisruptionBudget
Multi-AZ node placement
Topology spreading
Resource requests and limits
Monitoring
Automated rollback
```

## Q15. How do you select maxSurge and maxUnavailable?

I would consider:

- Application criticality
- Replica count
- Startup time
- Cluster capacity
- Pod resource requirements
- Availability requirements
- Graceful shutdown
- Traffic patterns

## Q16. When would you use Canary instead of RollingUpdate?

Canary is useful for high-risk deployments where the new version should initially receive only a small percentage of traffic.

Example:

```text
95% -> v1
5%  -> v2
```

If v2 is healthy:

```text
80% -> v1
20% -> v2
```

Eventually:

```text
100% -> v2
```

---

# 28. AWS/EKS Mapping

The same Kubernetes concepts apply when Kubernetes runs on Amazon EKS.

Typical architecture:

```text
Developer
    |
    v
CI/CD
    |
    v
Container Build
    |
    v
Amazon ECR
    |
    v
Amazon EKS
    |
    v
Deployment
    |
    v
ReplicaSet
    |
    +-- Pod
    +-- Pod
    +-- Pod
```

The Rolling Update itself is performed by the Kubernetes Deployment controller.

---

# 29. AWS Scenario — EKS Rolling Update

Suppose an EKS application is running:

```text
application:v1
```

and the new image is:

```text
application:v2
```

The Deployment can be updated:

```bash
kubectl set image deployment/application application=application:v2
```

Kubernetes then performs:

```text
Deployment
    |
    v
New ReplicaSet
    |
    v
New Pods
    |
    v
Readiness
    |
    v
Old Pods gradually removed
```

---

# 30. AWS Scenario — EKS Deployment Failure

If new Pods remain unavailable:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl rollout status deployment/<deployment-name>
```

Then investigate:

```text
Amazon ECR
EKS nodes
VPC
Security Groups
IAM
Load Balancer
DNS
CloudWatch
Application dependencies
```

Potential causes:

```text
ECR image pull failure
Incorrect image/tag
IAM permissions
Insufficient node capacity
Application startup failure
Readiness failure
Network connectivity
```

---

# 31. AWS Scenario — EKS Capacity During Rolling Update

Suppose:

```text
replicas = 10
```

Each Pod requests:

```text
CPU    = 500m
Memory = 1Gi
```

During a Rolling Update, `maxSurge` may temporarily create additional Pods.

Therefore cluster capacity must account for:

```text
Existing Pods
+
Temporary Surge Pods
```

If capacity is insufficient:

```text
Pod Status = Pending
```

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Insufficient cpu
Insufficient memory
```

Possible solutions:

- Add node capacity.
- Use node autoscaling.
- Increase node size.
- Optimize resource requests.
- Adjust rollout parameters.

---

# 32. AWS Scenario — EKS Control Plane vs Data Plane

A critical architectural distinction:

```text
Control Plane
    |
    +-- Kubernetes API
    +-- Scheduling
    +-- Cluster management

Data Plane
    |
    +-- Worker nodes
    +-- Pods
    +-- Application traffic
```

A control-plane/API problem does not automatically mean that already-running application Pods immediately stop serving traffic.

This distinction is important for resilient EKS architecture.

---

# 33. Real AWS Incident RCA — S3 2017

## Incident

In February 2017, AWS experienced a major Amazon S3 disruption in US-EAST-1.

AWS reported that an authorized operator was debugging an S3 billing-system problem and executed an established command intended to remove a small number of servers.

An incorrect command parameter resulted in more servers being removed than intended.

Some of the affected servers supported important S3 subsystems.

## Incident Chain

```text
Operational troubleshooting
        |
        v
Incorrect command parameter
        |
        v
Too many servers removed
        |
        v
Critical subsystem affected
        |
        v
Service degradation
        |
        v
Customer impact
```

## DevOps Lessons

### Limit Blast Radius

Production commands should have a limited scope.

### Validate Dangerous Operations

Use:

- Dry runs
- Pre-checks
- Approval
- Canary execution
- Least privilege

### Automation Requires Guardrails

Automation can reduce human error but can amplify errors without safety controls.

### Recovery Must Be Tested

Critical recovery procedures should be tested before an incident.

---

# 34. Real AWS Incident RCA — US-EAST-1 2021

## Incident

AWS experienced a major US-EAST-1 service event in December 2021.

AWS reported that automated scaling activity resulted in unexpected behavior involving a large number of internal network clients.

The resulting connection surge overwhelmed networking infrastructure, while retries increased the connection load.

## Incident Chain

```text
Automated scaling
        |
        v
Unexpected client behavior
        |
        v
Connection surge
        |
        v
Network congestion
        |
        v
Latency/errors
        |
        v
Retries
        |
        v
Additional load
```

## DevOps Lessons

Use:

- Exponential backoff
- Jitter
- Timeouts
- Bounded retries
- Circuit breakers where appropriate
- Rate limiting

A retry storm can turn a small failure into a large outage.

---

# 35. Real AWS Incident RCA — Kinesis 2024

## Incident

AWS reported a Kinesis Data Streams event in US-EAST-1 in July 2024.

AWS identified degradation within an internally used Kinesis cell.

The environment had an unusual workload distribution. During a deployment, workload was redistributed across hosts, resulting in resource contention.

## Incident Chain

```text
Routine deployment
        |
        v
Hosts temporarily removed
        |
        v
Workload redistribution
        |
        v
Uneven workload distribution
        |
        v
Resource contention
        |
        v
Service degradation
        |
        v
Dependent services affected
```

## Kubernetes Lessons

This maps closely to:

- Node scheduling
- Resource requests
- Resource limits
- Pod distribution
- Rolling Updates
- Node failure
- Pod rescheduling

Production testing should include deployment, node failure, rescheduling, scale-out, scale-in, uneven workloads, and high traffic.

---

# 36. Real AWS Incident RCA — Lambda 2023

## Incident

AWS reported a Lambda service event in US-EAST-1 in June 2023.

A capacity threshold exposed a latent software defect in the Lambda frontend fleet.

This resulted in increased invocation errors and latency and affected some dependent AWS services.

## Incident Chain

```text
Traffic increase
      |
      v
Lambda capacity expansion
      |
      v
Previously unseen threshold
      |
      v
Latent defect exposed
      |
      v
Invocation capacity problem
      |
      v
Errors and latency
      |
      v
Dependent services impacted
```

## DevOps Lessons

Scaling systems should be tested beyond normal operating ranges.

Test:

- Normal load
- Peak load
- Scale-out
- Scale-in
- Failure recovery
- Capacity boundaries

---

# 37. Production Incident RCA Template

## Incident Title

```text
Production Kubernetes Rolling Deployment Failure
```

## Severity

```text
SEV-1 / SEV-2 / SEV-3
```

## Impact

```text
Users affected:
Requests affected:
Duration:
Services affected:
```

## Detection

```text
Monitoring
Alert
Synthetic test
Customer report
```

## Timeline

```text
T0 - Deployment started
T1 - New Pods created
T2 - New Pods failed readiness
T3 - Alert triggered
T4 - Investigation started
T5 - Root cause identified
T6 - Rollback performed
T7 - Service recovered
```

## Root Cause

State the exact technical cause.

## Contributing Factors

Examples:

```text
Missing readiness probe
Incorrect image
Insufficient resources
Configuration error
Dependency failure
Insufficient monitoring
```

## Immediate Mitigation

Examples:

```text
Rollback
Scale capacity
Fix configuration
Stop deployment
```

## Corrective Actions

Technical changes required to prevent recurrence.

## Preventive Actions

Process, monitoring, architecture, or automation improvements.

---

# 38. Full DevOps Mock Interview

## Q1. Explain the task you performed.

### Answer

The task was to update an existing Kubernetes Deployment from `nginx:1.16` to `nginx:1.18` using a Rolling Update.

I first verified the Deployment and existing Pods.

I identified:

```text
Deployment = nginx-deployment
Container  = nginx-container
Old Image  = nginx:1.16
```

I then executed:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18
```

I monitored the update:

```bash
kubectl rollout status deployment/nginx-deployment
```

Finally, I verified:

```bash
kubectl get pods
kubectl get deployment nginx-deployment
```

and confirmed:

```text
3/3 Pods Ready
3/3 Pods Available
Image = nginx:1.18
```

---

## Q2. Why use a Deployment?

A Deployment provides:

- Replica management
- Rolling Updates
- Rollbacks
- Revision history
- Self-healing through ReplicaSets
- Declarative desired state

---

## Q3. What happens when the image changes?

The Deployment's Pod template changes.

Kubernetes creates a new ReplicaSet.

The new ReplicaSet creates Pods with the new image.

The old ReplicaSet is gradually scaled down.

```text
Deployment
   |
   +-- Old ReplicaSet
   |
   +-- New ReplicaSet
```

---

## Q4. How do you verify a successful rollout?

```bash
kubectl rollout status deployment/nginx-deployment
kubectl get deployment nginx-deployment
kubectl get pods
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Expected:

```text
Rollout successful
3/3 Ready
3 Available
All Pods Running
nginx:1.18
```

---

## Q5. What if the rollout gets stuck?

I would investigate:

```bash
kubectl get pods
kubectl describe deployment nginx-deployment
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

I would check for:

```text
ImagePullBackOff
CrashLoopBackOff
Pending
Readiness failure
Resource shortage
Configuration failure
Network failure
Dependency failure
```

If the new version is unsafe:

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

## Q6. What is the difference between Running and Ready?

`Running` means the container is running.

`Ready` means the Pod has passed its readiness condition and can receive traffic.

Therefore:

```text
Running != Ready
```

Example:

```text
READY   STATUS
0/1     Running
```

---

## Q7. Rolling Update vs Blue-Green?

### Rolling Update

```text
v1 v1 v1
   |
v1 v1 v2
   |
v1 v2 v2
   |
v2 v2 v2
```

### Blue-Green

```text
Blue:
v1 v1 v1

Green:
v2 v2 v2
```

Blue-Green requires additional capacity but can provide very fast rollback.

---

## Q8. Rolling Update vs Canary?

Rolling Update gradually replaces Pods.

Canary initially sends a small percentage of traffic to the new version.

```text
95% -> v1
5%  -> v2
```

If healthy:

```text
80% -> v1
20% -> v2
```

Eventually:

```text
100% -> v2
```

---

## Q9. How would you implement this in AWS?

For Amazon EKS:

```text
Git
 |
 v
CI/CD
 |
 v
Build Container
 |
 v
Security Scan
 |
 v
Amazon ECR
 |
 v
Amazon EKS
 |
 v
Deployment
 |
 v
Rolling Update
 |
 v
Readiness Validation
 |
 v
Monitoring
 |
 +---- Healthy -> Continue
 |
 +---- Failure -> Rollback
```

---

## Q10. What AWS services are involved?

Common services include:

```text
Amazon EKS   -> Kubernetes platform
Amazon ECR   -> Container registry
ALB/NLB      -> Load balancing
CloudWatch   -> Monitoring and logs
IAM          -> Access control
VPC          -> Networking
Route 53     -> DNS
CloudTrail   -> API auditing
Auto Scaling -> Capacity management
```

---

# 39. Production Deployment Architecture

```text
                    Git
                     |
                     v
                   CI/CD
                     |
          +----------+----------+
          |                     |
       Testing             Security Scan
          |                     |
          +----------+----------+
                     |
                     v
              Container Image
                     |
                     v
                Amazon ECR
                     |
                     v
                Amazon EKS
                     |
                Deployment
                     |
             RollingUpdate
                     |
          +----------+----------+
          |                     |
     Old ReplicaSet        New ReplicaSet
          |                     |
      Old Pods               New Pods
                                |
                           Readiness Check
                                |
                                v
                             Service
                                |
                                v
                         Load Balancer
                                |
                                v
                              Users
```

---

# 40. Production Safety Checklist

## Before Deployment

```text
[ ] Correct Deployment
[ ] Correct namespace
[ ] Correct container
[ ] Correct image
[ ] Correct image tag
[ ] Replica count verified
[ ] Existing Pods healthy
[ ] Sufficient cluster capacity
[ ] Readiness probe configured
[ ] Rollback plan available
```

## During Deployment

```text
[ ] Monitor rollout
[ ] Monitor Pod readiness
[ ] Monitor errors
[ ] Monitor latency
[ ] Monitor CPU
[ ] Monitor memory
[ ] Check application logs
[ ] Check deployment events
```

## After Deployment

```text
[ ] Deployment successfully rolled out
[ ] All replicas Ready
[ ] All replicas Available
[ ] All Pods Running
[ ] Correct image verified
[ ] Application health verified
[ ] Error rate normal
[ ] Latency normal
[ ] No unexpected restarts
```

---

# 41. Most Important Commands to Memorize

```bash
# Check Deployment
kubectl get deployment nginx-deployment

# Check Pods
kubectl get pods

# Check current image
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

# Check container name
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'

# Perform Rolling Update
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18

# Monitor rollout
kubectl rollout status deployment/nginx-deployment

# Check rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback
kubectl rollout undo deployment/nginx-deployment

# Deployment details
kubectl describe deployment nginx-deployment

# Pod details
kubectl describe pod <pod-name>

# Pod logs
kubectl logs <pod-name>

# Kubernetes events
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 42. One-Minute Interview Answer

> A Rolling Update is the Kubernetes Deployment strategy used to gradually replace old application Pods with new Pods. In this task, the existing `nginx-deployment` had three replicas running `nginx:1.16`. I identified the container as `nginx-container` and updated the Deployment using `kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18`. Kubernetes created a new ReplicaSet and gradually replaced the old Pods. I monitored the rollout using `kubectl rollout status deployment/nginx-deployment`. After completion, I verified that the Deployment had 3/3 Ready and 3/3 Available replicas, all Pods were Running, and the Deployment was using `nginx:1.18`. In production, I would also use readiness probes, resource requests and limits, PodDisruptionBudgets, appropriate maxSurge/maxUnavailable values, monitoring, and rollback or progressive delivery mechanisms.

---

# 43. Final Mental Model

```text
New Application Version
        |
        v
Update Deployment
        |
        v
New ReplicaSet
        |
        v
New Pods
        |
        v
Readiness Checks
        |
        v
New Pods Ready
        |
        v
Old Pods Gradually Removed
        |
        v
All Desired Replicas Available
        |
        v
Rollout Successful
```

If the new version fails:

```text
New Version
     |
     v
Failure
     |
     v
Investigate
     |
     v
Rollback
     |
     v
Previous Version
```

---

# 44. Core DevOps Takeaway

```text
Rolling Update
=
Controlled version change
+
Gradual Pod replacement
+
Application availability
+
Health validation
+
Rollback capability
```

The most important principle is:

> Never consider a deployment successful merely because Pods are `Running`. Validate rollout status, Pod readiness, replica availability, application health, error rate, and the actual image/version running in the Deployment.

---

# 45. Official AWS Incident Sources

AWS S3 2017 Post-Event Summary:
https://aws.amazon.com/message/41926/

AWS US-EAST-1 December 2021 Post-Event Summary:
https://aws.amazon.com/message/12721/

AWS Kinesis July 2024 Post-Event Summary:
https://aws.amazon.com/jp/message/073024/

AWS Lambda June 2023 Post-Event Summary:
https://aws.amazon.com/message/061323/
