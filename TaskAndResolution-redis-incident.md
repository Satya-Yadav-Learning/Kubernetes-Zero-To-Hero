# TaskAndResolution.md

## Incident Summary

DevOps team had a **Redis application** running successfully on a Kubernetes cluster. A recent configuration change introduced errors, causing the Redis application to go down. The Pods were stuck in a non-running state. The objective was to **identify the root cause, fix the configuration, and restore Redis service as quickly as possible**.


All actions below were performed from the **host**.

---

## Impact

* Redis Pods were not reaching `Running` state
* Application dependent on Redis was unavailable
* Deployment showed `MinimumReplicasUnavailable`

---

## Step 1: Identify the Issue

### Check Pod Status

```bash
kubectl get pods
```

### Output

```text
redis-deployment-6fd9d5fcb-b8hgv   0/1   ContainerCreating   0   83s
```

### Interpretation

* Pod stuck in `ContainerCreating`
* Indicates image pull, volume mount, or node-level issue (not application crash)

---

## Step 2: Describe the Pod (Root Cause Analysis)

```bash
kubectl describe pod redis-deployment-6fd9d5fcb-b8hgv
```

### Key Events

```text
FailedMount
MountVolume.SetUp failed for volume "config" : configmap "redis-cofig" not found
```

### Findings

* Pod failed before container startup
* Volume mount error due to missing ConfigMap

---

## Step 3: Inspect Deployment Configuration

```bash
kubectl describe deployment redis-deployment
```

### Observed Issues

```yaml
image: redis:alpin          # invalid image tag
configMap:
  name: redis-cofig         # misspelled ConfigMap
```

### Problems Identified

1. **Invalid Redis image tag** (`redis:alpin`)
2. **Incorrect ConfigMap name** (`redis-cofig` instead of `redis-config`)

Either issue alone could break the Pod; together they guaranteed failure.

---

## Step 4: Apply the Fix

### Edit the Deployment

```bash
kubectl edit deployment redis-deployment
```

### Corrected Container Configuration

```yaml
containers:
- name: redis-container
  image: redis:alpine
  ports:
  - containerPort: 6379
  resources:
    requests:
      cpu: 300m
```

### Cleanup Action

* Removed invalid ConfigMap volume reference
* Redis does not require a ConfigMap to start

---

## Step 5: Kubernetes Self-Healing in Action

After saving the deployment:

* A new ReplicaSet was created
* Old broken Pod was terminated
* New Pod launched automatically

---

## Step 6: Verify Recovery

### Pod Status

```bash
kubectl get pods
```

### Output

```text
redis-deployment-6458785968-kf2j9   1/1   Running   0   25s
```

---

## Step 7: Validate Redis Application

```bash
kubectl logs redis-deployment-6458785968-kf2j9
```

### Key Log Output

```text
Ready to accept connections tcp
```

### Confirmation

* Redis initialized successfully
* Application is ready to serve traffic

---

## Final Root Cause & Resolution

### Root Cause

* Typo in Redis image name
* Typo in ConfigMap reference
* Pod blocked during volume mount phase

### Resolution

* Corrected Redis image to `redis:alpine`
* Removed faulty ConfigMap dependency
* Allowed Kubernetes rolling update to restore service

---

## Scenario-Based Interview Questions & Answers

### L1 – Junior DevOps Engineer

**Q1. Why was the Pod stuck in `ContainerCreating`?**
Because Kubernetes could not mount a required volume due to a missing ConfigMap.

**Q2. What command helps identify this issue?**
`kubectl describe pod` — especially the Events section.

---

### L2 – Mid-Level DevOps Engineer

**Q3. Why didn’t the Pod enter `CrashLoopBackOff`?**
The container never started; the failure happened before runtime during volume setup.

**Q4. Why was Redis not restarting repeatedly?**
Because the container never launched; restart logic applies only after container start.

---

### L3 – Senior DevOps Engineer

**Q5. How could this incident have been prevented?**

* YAML validation
* Pre-deployment checks
* Avoiding unnecessary ConfigMap dependencies

**Q6. Why remove the ConfigMap instead of fixing it?**
Redis does not require it for default operation; removing reduced risk and recovery time.

---

### L4 – Architect Level

**Q7. How would you design Redis for production?**

* StatefulSet instead of Deployment
* Persistent Volumes
* Readiness & liveness probes
* Sentinel or managed Redis

---

## AWS EKS – Real-World Outage Mapping (Scenario-Based RCA)

### Scenario 1: Pod Stuck in `ContainerCreating` on EKS

**Root Cause:**
ConfigMap or Secret missing due to namespace mismatch.

**Resolution:**
Validated resource existence and corrected references.

---

### Scenario 2: Application Down After Image Update

**Root Cause:**
Incorrect image tag pushed to ECR.

**Resolution:**
Rolled back Deployment and enforced image tag validation.

---

### Scenario 3: Redis Pod Restarting Frequently

**Root Cause:**
OOMKilled due to insufficient memory limits.

**Resolution:**
Adjusted resource limits and enabled monitoring via CloudWatch Container Insights.
[O
---

## Preventive Measures (Best Practices)

* Avoid typos via CI linting (kubeval, kube-score)
* Use GitOps for controlled changes
* Implement admission controllers (OPA/Kyverno)
* Prefer managed Redis where possible (ElastiCache)

---

## Final Conclusion

This incident demonstrates how small configuration errors can cause application downtime in Kubernetes. By methodically inspecting Pod events and Deployment specifications, the root cause was identified and fixed. Kubernetes’ self-healing capabilities ensured rapid recovery once the configuration was corrected.

