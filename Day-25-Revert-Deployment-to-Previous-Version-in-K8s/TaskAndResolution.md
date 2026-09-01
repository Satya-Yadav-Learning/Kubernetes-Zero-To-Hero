# TaskAndResolution.md

# Kubernetes Deployment Rollback — Complete Task & Interview Handbook

---

# 1. Task Objective

## Requirement

A new release of an application deployed through Kubernetes was found to have a customer-impacting bug.

The requirement was to roll back the Kubernetes Deployment named:

```text
nginx-deployment
```

to its **previous revision**.

The Kubernetes cluster was already configured and accessible through `kubectl`.

---

# 2. Kubernetes Rollback Concept

A Kubernetes Deployment maintains revision history when its Pod template changes.

Example:

```text
Revision 1
    |
    | Original application
    v
Revision 2
    |
    | New release with bug
    v
Rollback
    |
    v
Revision 1
```

The command:

```bash
kubectl rollout undo deployment/nginx-deployment
```

tells Kubernetes to restore the Deployment to its previous revision.

---

# 3. Verify the Deployment

## Command

```bash
kubectl get deployment nginx-deployment
```

## Output

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2m13s
```

## Explanation

The Deployment was running three replicas.

```text
READY       = 3/3
UP-TO-DATE  = 3
AVAILABLE   = 3
```

This means all three currently desired Pods were ready and available.

---

# 4. Check Deployment Revision History

## Command

```bash
kubectl rollout history deployment/nginx-deployment
```

## Output

```text
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine --record=true
```

## Explanation

The Deployment had two revisions.

```text
Revision 1
    |
    +-- Previous version

Revision 2
    |
    +-- New release
    +-- nginx:alpine
```

The task required returning to the previous revision.

Therefore the required target was:

```text
Revision 1
```

---

# 5. Execute the Rollback

## Command

```bash
kubectl rollout undo deployment/nginx-deployment
```

## Output

```text
deployment.apps/nginx-deployment rolled back
```

## Explanation

This command instructs Kubernetes to roll the Deployment back to its previous revision.

The command does not require a revision number because the requirement is specifically to return to the immediately previous revision.

Kubernetes creates the appropriate ReplicaSet/Pod state for the previous Deployment revision and gradually replaces the current Pods.

---

# 6. Monitor the Rollback

## Command

```bash
kubectl rollout status deployment/nginx-deployment
```

## Output

```text
deployment "nginx-deployment" successfully rolled out
```

## Explanation

This confirms that the rollback operation completed successfully.

The important word is:

```text
successfully rolled out
```

It indicates that the Deployment controller successfully reached the desired state.

---

# 7. Verify Pods

## Command

```bash
kubectl get pods
```

## Output

```text
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-fc677cbc9-fgcp5   1/1     Running   0          90s
nginx-deployment-fc677cbc9-lgw5m   1/1     Running   0          91s
nginx-deployment-fc677cbc9-tz84k   1/1     Running   0          92s
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

The Pod names have been retained only as representative/anonymized environment-specific identifiers.

---

# 8. Verify Deployment After Rollback

## Command

```bash
kubectl get deployment nginx-deployment
```

## Output

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           6m20s
```

## Explanation

The Deployment has:

```text
READY       = 3/3
UP-TO-DATE  = 3
AVAILABLE   = 3
```

This confirms that all desired replicas are available after the rollback.

---

# 9. Verify the Actual Image After Rollback

## Command

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Output

```text
nginx:1.16
```

## Explanation

The final image is:

```text
nginx:1.16
```

This confirms that the Deployment has returned to the previous application version.

The revision history showed that Revision 2 used:

```text
nginx:alpine
```

After the rollback, the Deployment is again using:

```text
nginx:1.16
```

Therefore the rollback was successful.

---

# 10. Final Verification

The final state was:

```text
Deployment:
nginx-deployment

Previous Revision:
Revision 1

New/Buggy Revision:
Revision 2

Previous Image:
nginx:1.16

New Release Image:
nginx:alpine

Rollback:
Successful

Rollout:
Successfully rolled out

Pods:
3/3 Running

Deployment:
3/3 Ready

Available:
3/3
```

## Final Result

```text
TASK COMPLETED SUCCESSFULLY
```

---

# 11. Complete Correct Command Sequence

The complete successful sequence for this task was:

```bash
# 1. Check Deployment
kubectl get deployment nginx-deployment

# 2. Check revision history
kubectl rollout history deployment/nginx-deployment

# 3. Roll back to the previous revision
kubectl rollout undo deployment/nginx-deployment

# 4. Monitor rollback
kubectl rollout status deployment/nginx-deployment

# 5. Verify Pods
kubectl get pods

# 6. Verify Deployment
kubectl get deployment nginx-deployment

# 7. Verify the image
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Expected final image:

```text
nginx:1.16
```

---

# 12. How Kubernetes Rollback Works Internally

A Deployment manages ReplicaSets.

```text
Deployment
    |
    +----------------------+
    |                      |
    v                      v
Old ReplicaSet         New ReplicaSet
    |                      |
    v                      v
Old Pods                New Pods
```

When a Deployment is updated:

```text
Old Version
nginx:1.16
     |
     v
New Version
nginx:alpine
```

Kubernetes creates a new ReplicaSet.

If the new release has a problem:

```text
nginx:alpine
     |
     | Rollback
     v
nginx:1.16
```

Kubernetes restores the previous Pod template.

---

# 13. Deployment → ReplicaSet → Pod

The hierarchy is:

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

## Deployment

Responsible for:

- Desired state
- Replica management
- Rolling Updates
- Rollbacks
- Revision history

## ReplicaSet

Responsible for maintaining the required number of Pods.

## Pod

Provides the execution environment for containers.

## Container

Runs the actual application process.

---

# 14. Why Pod Names Changed

When a Deployment revision changes, Kubernetes normally creates a new ReplicaSet.

ReplicaSets use a Pod-template hash.

Therefore Pods created by different revisions normally have different names.

Conceptually:

```text
Old ReplicaSet
    |
    +-- application-oldhash-pod1
    +-- application-oldhash-pod2
    +-- application-oldhash-pod3
```

New revision:

```text
New ReplicaSet
    |
    +-- application-newhash-pod1
    +-- application-newhash-pod2
    +-- application-newhash-pod3
```

After rollback, Kubernetes can again use the ReplicaSet corresponding to the previous revision.

---

# 15. Rolling Update vs Rollback

## Rolling Update

Used when introducing a new version.

```text
v1
 |
 v
v2
```

Example:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18
```

## Rollback

Used when the new version causes a problem.

```text
v2
 |
 v
v1
```

Example:

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

# 16. Rollout History

Use:

```bash
kubectl rollout history deployment/nginx-deployment
```

Example:

```text
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine --record=true
```

Revision numbers identify Deployment revisions.

The revision itself is not necessarily the same thing as an application version.

For example:

```text
Revision 1 → nginx:1.16
Revision 2 → nginx:alpine
```

---

# 17. Rollback to a Specific Revision

If a specific historical revision must be restored rather than simply the previous revision:

```bash
kubectl rollout history deployment/nginx-deployment
```

Then:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=<revision-number>
```

Example:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

For this task, however, the correct command was simply:

```bash
kubectl rollout undo deployment/nginx-deployment
```

because the requirement was to return to the previous revision.

---

# 18. Rollback Verification Strategy

Never verify a rollback using only one command.

Use multiple layers of validation.

## Layer 1 — Rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected:

```text
deployment "nginx-deployment" successfully rolled out
```

## Layer 2 — Pods

```bash
kubectl get pods
```

Expected:

```text
1/1 Running
```

for each required Pod.

## Layer 3 — Deployment

```bash
kubectl get deployment nginx-deployment
```

Expected:

```text
3/3
3
3
```

## Layer 4 — Application Image

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Expected:

```text
nginx:1.16
```

This four-layer verification provides much stronger evidence that the rollback succeeded.

---

# 19. Important Difference: Running vs Ready

A Pod showing:

```text
STATUS = Running
```

does not automatically mean that it is ready to receive traffic.

Example:

```text
READY   STATUS
0/1     Running
```

The container is running, but it may not have passed its readiness check.

Therefore production validation should consider:

```text
Running
+
Ready
+
Available
+
Application health
```

---

# 20. What If Rollback Gets Stuck?

Start with:

```bash
kubectl rollout status deployment/nginx-deployment
```

Then inspect Pods:

```bash
kubectl get pods
```

Describe the Deployment:

```bash
kubectl describe deployment nginx-deployment
```

Describe the affected Pod:

```bash
kubectl describe pod <pod-name>
```

Check logs:

```bash
kubectl logs <pod-name>
```

Check events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common problems include:

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
Network failure
Dependency failure
```

---

# 21. What If the Previous Image Cannot Be Pulled?

A rollback can fail if the previous image is no longer available.

For example:

```text
Previous image:
application:v1
```

If the registry no longer contains the image, Kubernetes may produce:

```text
ErrImagePull
ImagePullBackOff
```

Production best practice:

- Keep required production image versions available.
- Use immutable image tags or digests.
- Avoid deleting images that active rollback procedures depend on.
- Maintain a controlled image-retention policy.

---

# 22. Why Immutable Image Tags Matter

Using:

```text
nginx:1.16
```

is better than using:

```text
nginx:latest
```

for controlled production releases.

Even better for strict reproducibility is an image digest:

```text
image@sha256:<digest>
```

Conceptually:

```text
Tag
  |
  v
Image
  |
  v
Digest
```

A digest identifies a specific image artifact.

This improves deployment reproducibility and rollback reliability.

---

# 23. Scenario-Based Interview Questions — L1

## Q1. What was the task?

### Answer

The task was to roll back the Kubernetes Deployment `nginx-deployment` to its previous revision because the latest application release had a customer-reported bug.

I first checked the Deployment and rollout history, then executed:

```bash
kubectl rollout undo deployment/nginx-deployment
```

I monitored the rollback and verified that all three Pods were running and that the image had returned to `nginx:1.16`.

---

## Q2. What command performs a rollback?

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

## Q3. How do you check Deployment history?

```bash
kubectl rollout history deployment/nginx-deployment
```

---

## Q4. How do you monitor rollback progress?

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## Q5. How do you verify the Pods?

```bash
kubectl get pods
```

Look for:

```text
READY   STATUS
1/1     Running
```

---

## Q6. How do you verify the application image?

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# 24. Scenario-Based Interview Questions — L2

## Q7. What is a Kubernetes Deployment?

A Deployment is a Kubernetes controller used to manage application workloads declaratively.

It provides:

- Replica management
- Rolling Updates
- Rollbacks
- Revision history
- Self-healing through ReplicaSets

---

## Q8. What happens during a rollback?

The Deployment restores a previous Pod template.

Kubernetes then reconciles the desired state by creating or scaling the appropriate ReplicaSet and Pods.

Conceptually:

```text
Current Version
      |
      v
Rollback Requested
      |
      v
Previous Pod Template
      |
      v
Previous ReplicaSet
      |
      v
Previous Pods
```

---

## Q9. Why use `kubectl rollout undo` instead of manually deleting Pods?

Deleting Pods does not necessarily restore the previous application version.

The Deployment controller will simply recreate Pods according to the current Deployment specification.

Rollback changes the Deployment's desired Pod template itself.

Therefore:

```text
Delete Pod
    ≠
Rollback Application
```

---

## Q10. What is a ReplicaSet?

A ReplicaSet ensures that the desired number of Pods exists.

For example:

```text
replicas = 3
```

The ReplicaSet continuously attempts to maintain three matching Pods.

---

## Q11. What is a Deployment revision?

A Deployment revision represents a version of the Deployment's Pod template.

Example:

```text
Revision 1 → nginx:1.16
Revision 2 → nginx:alpine
```

A rollback can restore an earlier revision.

---

# 25. Scenario-Based Interview Questions — L3

## Q12. A rollback command succeeded, but users still see errors. What do you check?

I would not assume the rollback solved the problem merely because the command succeeded.

I would check:

```bash
kubectl rollout status deployment/nginx-deployment
kubectl get pods
kubectl get deployment nginx-deployment
kubectl get svc
kubectl get endpoints
kubectl logs <pod-name>
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Then verify:

- Correct image
- Pod readiness
- Service endpoints
- Load balancer
- Ingress
- Application logs
- Network connectivity
- External dependencies
- Database connectivity
- DNS
- Error rates
- Latency

---

## Q13. The rollback completed but one Pod is `CrashLoopBackOff`. What do you do?

I would identify the affected Pod:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

and:

```bash
kubectl logs <pod-name>
```

I would inspect the previous container logs if appropriate:

```bash
kubectl logs <pod-name> --previous
```

Then determine whether the cause is:

```text
Application failure
Configuration error
Environment variable
Secret/ConfigMap
Resource limitation
Dependency failure
Image problem
Startup failure
```

---

## Q14. How do you know the rollback actually restored the previous version?

I would verify multiple layers.

First:

```bash
kubectl rollout status deployment/nginx-deployment
```

Then:

```bash
kubectl get pods
```

Then:

```bash
kubectl get deployment nginx-deployment
```

Finally:

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

In this task the final image was:

```text
nginx:1.16
```

---

## Q15. What is the difference between `kubectl rollout undo` and `kubectl delete pod`?

### `kubectl rollout undo`

Changes the Deployment back to an earlier revision.

```text
Application version changes
```

### `kubectl delete pod`

Deletes a specific Pod.

The Deployment/ReplicaSet can recreate it using the same current specification.

```text
Pod instance changes
```

Therefore they solve different problems.

---

# 26. Scenario-Based Interview Questions — L4

## Q16. How would you design a production rollback strategy?

I would design rollback around:

```text
Immutable images
Versioned artifacts
Deployment revision history
Health checks
Readiness probes
Automated monitoring
Clear rollback thresholds
Fast rollback procedure
Observability
Change auditing
Database compatibility
Backward-compatible APIs
```

The key principle is that rollback must be safe not only for application code but also for data and dependencies.

---

## Q17. Why can application rollback be difficult when database schema changes are involved?

Suppose version 2 changes the database schema:

```text
v1 → Old schema
v2 → New schema
```

If the application is rolled back:

```text
v2 → v1
```

the old application may not understand the new database schema.

Therefore production deployments should use backward-compatible database migrations where possible.

A safer pattern is:

```text
Expand
  |
  v
Deploy compatible application
  |
  v
Migrate usage
  |
  v
Contract
```

This is commonly called an **expand-and-contract migration strategy**.

---

## Q18. What if the application rollback is successful but the database migration is irreversible?

The Kubernetes Deployment can be rolled back, but the database may not be safely reversible.

Therefore I would:

1. Stop further rollout.
2. Assess database impact.
3. Restore application compatibility if possible.
4. Follow the database recovery procedure.
5. Use backups/PITR when required.
6. Validate data integrity.
7. Communicate customer impact.

The important architectural lesson is:

```text
Application rollback
        ≠
Complete system rollback
```

---

## Q19. How would you implement automated rollback?

A production pipeline could use:

```text
Git
 |
 v
CI/CD
 |
 v
Build
 |
 v
Security Scan
 |
 v
Registry
 |
 v
Kubernetes Deployment
 |
 v
Rolling/Canary Release
 |
 v
Health Validation
 |
 +---- Healthy ----> Continue
 |
 +---- Unhealthy --> Rollback
```

Health signals could include:

- HTTP error rate
- Latency
- Availability
- Pod readiness
- Container restarts
- Application-specific metrics
- Business KPIs

---

## Q20. When would you use Canary instead of immediate rollback?

If the new version has only been exposed to a small percentage of traffic, Canary deployment allows us to stop promotion before the entire fleet is affected.

Example:

```text
95% → Version 1
5%  → Version 2
```

If Version 2 fails:

```text
100% → Version 1
```

This can reduce blast radius.

---

# 27. AWS/EKS Mapping

The same Kubernetes rollback mechanism can be used on Amazon EKS.

Architecture:

```text
Developer
    |
    v
Git Repository
    |
    v
CI/CD Pipeline
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
Kubernetes Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

Rollback:

```text
Deployment
    |
    v
Current Revision
    |
    | kubectl rollout undo
    v
Previous Revision
```

---

# 28. AWS Scenario — EKS Rollback

Suppose an application in EKS was deployed with:

```text
application:v2
```

and customers report a problem.

The previous version is:

```text
application:v1
```

The Kubernetes rollback command is:

```bash
kubectl rollout undo deployment/application
```

Then:

```bash
kubectl rollout status deployment/application
```

Verify:

```bash
kubectl get pods
```

And verify the image:

```bash
kubectl get deployment application -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# 29. AWS Production Rollback Architecture

```text
                    Git
                     |
                     v
                    CI
                     |
                     v
                  Build
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
             Progressive Release
                     |
          +----------+----------+
          |                     |
       Healthy                Failed
          |                     |
          v                     v
      Continue              Rollback
                                |
                                v
                         Previous Version
```

---

# 30. AWS Services Commonly Involved

## Amazon EKS

Runs the Kubernetes workloads.

## Amazon ECR

Stores container images.

## IAM

Controls permissions.

## VPC

Provides networking.

## Elastic Load Balancing

Provides traffic distribution.

## Amazon CloudWatch

Provides monitoring, metrics, and logs where configured.

## AWS CloudTrail

Provides API activity auditing.

## Route 53

Provides DNS services.

---

# 31. AWS/EKS Rollback Incident Scenario

## Situation

A new application version is deployed to EKS.

Shortly afterward:

```text
HTTP 5xx ↑
Latency ↑
Customer errors ↑
```

## Response

First check:

```bash
kubectl rollout status deployment/application
kubectl get pods
kubectl describe deployment application
kubectl logs <pod-name>
```

Check application metrics and AWS monitoring.

If the new version is confirmed as the cause:

```bash
kubectl rollout undo deployment/application
```

Then:

```bash
kubectl rollout status deployment/application
```

Finally verify:

```bash
kubectl get pods
```

and:

```bash
kubectl get deployment application -o jsonpath='{.spec.template.spec.containers[0].image}'
```

---

# 32. Production RCA Example — Bad Application Release

## Incident

A new application release caused elevated HTTP 5xx responses.

## Impact

```text
Customer requests failed.
Error rate increased.
Application availability decreased.
```

## Detection

Monitoring detected:

```text
HTTP 5xx increase
Latency increase
```

## Timeline

```text
T0  → New version deployed
T1  → New Pods became available
T2  → Error rate increased
T3  → Monitoring alert triggered
T4  → Application logs inspected
T5  → New release identified as probable cause
T6  → Deployment rollback initiated
T7  → Previous Pods became available
T8  → Error rate returned to normal
```

## Root Cause

A defect introduced in the new application release caused production request failures.

## Immediate Mitigation

Rollback to the previous known-good Deployment revision.

```bash
kubectl rollout undo deployment/application
```

## Verification

```bash
kubectl rollout status deployment/application
kubectl get pods
```

Application metrics were then checked to confirm recovery.

## Corrective Actions

- Add stronger pre-production testing.
- Add automated deployment health checks.
- Add progressive delivery.
- Improve application-level monitoring.
- Define automatic rollback thresholds.
- Maintain immutable release artifacts.

---

# 33. Production RCA — Failed Rollback

## Scenario

The rollback command is issued, but Pods cannot start.

## Investigation

```bash
kubectl get pods
```

Shows:

```text
ImagePullBackOff
```

Then:

```bash
kubectl describe pod <pod-name>
```

Possible root cause:

```text
Previous container image no longer exists in the registry.
```

## Root Cause

The organization deleted an image that was required for rollback.

## Corrective Actions

- Implement image retention policies.
- Protect production image versions.
- Use immutable image digests.
- Test rollback procedures.
- Maintain disaster-recovery procedures.
- Do not remove artifacts required by active releases.

---

# 34. Production RCA — Rollback Did Not Fix Customer Issue

## Scenario

The Deployment successfully rolled back, but customers still report errors.

## Investigation

Possible architecture:

```text
Customer
   |
   v
DNS
   |
   v
Load Balancer
   |
   v
Ingress
   |
   v
Service
   |
   v
Pods
   |
   v
Application
   |
   v
Database / External Services
```

The Deployment may not have been the root cause.

Investigate:

```text
Load Balancer
Ingress
Service
DNS
Network policies
Database
Cache
External APIs
Authentication
Application dependencies
```

## Lesson

A successful Kubernetes rollback proves that the Deployment reverted.

It does not prove that the entire application ecosystem has recovered.

---

# 35. DevOps Mock Interview — Junior

## Q1. What command did you use for rollback?

### Answer

```bash
kubectl rollout undo deployment/nginx-deployment
```

This rolls the Deployment back to its previous revision.

---

## Q2. How did you verify the rollback?

### Answer

I used:

```bash
kubectl rollout status deployment/nginx-deployment
```

Then:

```bash
kubectl get pods
```

Then:

```bash
kubectl get deployment nginx-deployment
```

Finally I verified the image:

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

The image was:

```text
nginx:1.16
```

---

## Q3. What was the final Pod state?

### Answer

All three Pods were:

```text
READY   = 1/1
STATUS  = Running
```

Therefore the application had three operational Pods.

---

# 36. DevOps Mock Interview — Intermediate

## Q4. Why didn't you simply delete the new Pods?

### Answer

Deleting Pods would not change the Deployment's desired state.

The ReplicaSet would recreate Pods using the current, potentially broken, Deployment specification.

A rollback changes the Deployment back to the previous revision.

---

## Q5. What is the difference between a ReplicaSet and Deployment?

### Answer

A ReplicaSet maintains the desired number of Pods.

A Deployment manages ReplicaSets and provides higher-level functionality such as:

```text
Rolling Updates
Rollbacks
Revision history
Replica management
```

---

## Q6. What would you check if the rollback gets stuck?

### Answer

I would use:

```bash
kubectl get pods
kubectl describe deployment nginx-deployment
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Then investigate the specific failure.

---

# 37. DevOps Mock Interview — Senior

## Q7. Would you automatically rollback every failed deployment?

### Answer

No.

I would define rollback criteria based on measurable health signals.

Examples:

```text
HTTP 5xx rate
Latency
Pod readiness
Container restarts
Application health
Business transaction success rate
```

A single transient error should not necessarily trigger an automated rollback.

---

## Q8. How would you reduce rollback time?

### Answer

I would use:

```text
Immutable container images
Fast CI/CD pipelines
Deployment history
Automated health checks
Progressive delivery
Automated rollback
Well-tested runbooks
Observability
Predefined rollback criteria
```

---

## Q9. What is the risk of database migrations during rollback?

### Answer

The application can be rolled back while the database schema remains changed.

If the old application cannot work with the new schema, rollback may create another outage.

Therefore database changes should preferably be backward compatible.

---

# 38. DevOps Mock Interview — Architect

## Q10. Design a resilient deployment and rollback architecture.

### Answer

I would use:

```text
Git
 |
 v
CI/CD
 |
 +--> Unit Tests
 |
 +--> Integration Tests
 |
 +--> Security Scan
 |
 v
Immutable Container
 |
 v
Amazon ECR
 |
 v
Amazon EKS
 |
 v
Canary / Rolling Deployment
 |
 v
Readiness + Liveness + Startup Checks
 |
 v
Metrics
 |
 +---- Healthy ----> Progressive Promotion
 |
 +---- Unhealthy --> Automated Rollback
 |
 v
Previous Known-Good Version
```

I would also ensure:

```text
Multi-AZ worker capacity
PodDisruptionBudget
Topology spreading
Resource requests/limits
Image retention
Immutable artifacts
Observability
Audit logging
Database compatibility
Disaster recovery
```

---

# 39. Architect-Level Question — Rollback Strategy

## Question

What would you consider before implementing automatic rollback?

## Answer

I would consider:

### Application Health

```text
HTTP status
Latency
Error rate
Business KPIs
```

### Kubernetes Health

```text
Pod readiness
Pod restarts
Replica availability
Deployment status
```

### Infrastructure

```text
Node capacity
Network
Load Balancer
Storage
```

### Dependencies

```text
Database
Cache
External APIs
Messaging
DNS
```

### Data Compatibility

The previous application version must remain compatible with the current data schema.

---

# 40. Important Production Principle

A rollback should be treated as a controlled production change.

```text
Detect
  |
  v
Assess
  |
  v
Confirm Release Correlation
  |
  v
Rollback
  |
  v
Validate Kubernetes
  |
  v
Validate Application
  |
  v
Validate Customer Experience
  |
  v
Close Incident
```

Do not stop at:

```text
kubectl rollout undo
```

A successful command is not the same as a successful incident recovery.

---

# 41. Most Important Commands to Memorize

```bash
# Check Deployment
kubectl get deployment nginx-deployment

# Check rollout history
kubectl rollout history deployment/nginx-deployment

# Roll back to previous revision
kubectl rollout undo deployment/nginx-deployment

# Monitor rollback
kubectl rollout status deployment/nginx-deployment

# Check Pods
kubectl get pods

# Check Deployment
kubectl get deployment nginx-deployment

# Verify current image
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'

# Describe Deployment
kubectl describe deployment nginx-deployment

# Describe Pod
kubectl describe pod <pod-name>

# Check application logs
kubectl logs <pod-name>

# Check previous container logs
kubectl logs <pod-name> --previous

# Check Kubernetes events
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 42. One-Minute Interview Answer

> A customer reported a bug in the latest release, so I needed to roll back the Kubernetes Deployment `nginx-deployment` to its previous revision. I first verified the Deployment and checked its rollout history using `kubectl rollout history deployment/nginx-deployment`. The history showed Revision 1 as the previous version and Revision 2 as the latest release. I then executed `kubectl rollout undo deployment/nginx-deployment`, which rolled the Deployment back to the previous revision. I monitored the operation using `kubectl rollout status deployment/nginx-deployment`, which reported that the Deployment successfully rolled out. I then verified the Pods using `kubectl get pods` and confirmed that all three Pods were `1/1 Running`. Finally, I verified the Deployment image using `kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'`, which returned `nginx:1.16`. Therefore, the rollback was successfully completed and the application was restored to the previous version.

---

# 43. Quick Revision Sheet

## Rollback

```bash
kubectl rollout undo deployment/nginx-deployment
```

## History

```bash
kubectl rollout history deployment/nginx-deployment
```

## Status

```bash
kubectl rollout status deployment/nginx-deployment
```

## Pods

```bash
kubectl get pods
```

## Deployment

```bash
kubectl get deployment nginx-deployment
```

## Image

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Rollback to Specific Revision

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=<revision-number>
```

---

# 44. Final Mental Model

```text
Customer Reports Bug
        |
        v
Identify Problematic Release
        |
        v
Check Deployment History
        |
        v
kubectl rollout undo
        |
        v
Kubernetes Restores Previous Revision
        |
        v
Monitor Rollout
        |
        v
Verify Pods
        |
        v
Verify Deployment
        |
        v
Verify Image
        |
        v
Validate Application
        |
        v
Recovery Confirmed
```

---

# 45. Core DevOps Takeaways

```text
Deployment
    |
    +--> Maintains application desired state
    |
    +--> Manages ReplicaSets
    |
    +--> Supports Rolling Updates
    |
    +--> Maintains revision history
    |
    +--> Supports Rollback
```

The most important rollback command is:

```bash
kubectl rollout undo deployment/nginx-deployment
```

The most important verification commands are:

```bash
kubectl rollout status deployment/nginx-deployment
kubectl get pods
kubectl get deployment nginx-deployment
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

For this task, the final verified state was:

```text
Deployment       = nginx-deployment
Previous Image   = nginx:1.16
Rollback         = Successful
Rollout          = Successfully rolled out
Pods             = 3/3 Running
Deployment       = 3/3 Ready
Available        = 3/3
```

```text
TASK COMPLETED SUCCESSFULLY
```
