# Scenario-Based Interview Questions & Answers (Rolling Update – nginx Deployment)

---

This document contains **scenario-based Kubernetes interview questions and answers** derived directly from the **rolling update task** performed on the `nginx-deployment` (image update from `nginx:1.16` to `nginx:1.19`) in the *notecloud* Kubernetes cluster.

The questions are structured from **L1 (Junior)** to **L4 (Architect)** and reflect **real production behavior, troubleshooting steps, and design decisions**.

---

## 🟢 L1 – Junior Level (Core Concepts)

### Scenario 1: Updating an application without downtime

**Question:**
An nginx application is running with three Pods. A new image version `nginx:1.19` needs to be deployed without stopping the application. Which Kubernetes object supports this and how?

**Answer:**
A **Deployment** supports this using a **rolling update** strategy.

**Explanation:**

* Deployments manage ReplicaSets
* During a rolling update, new Pods are created gradually
* Old Pods are terminated only after new Pods become ready
* This ensures continuous availability

---

### Scenario 2: Image update command fails

**Question:**
You executed `kubectl set image deployment/nginx-deployment nginx:1.19` and received an error. Why did this happen?

**Answer:**
Because Kubernetes requires the format **`container-name=image`** when updating images.

**Explanation:**

* Kubernetes updates containers, not images directly
* The container name must be explicitly specified

---

### Scenario 3: Verifying the update

**Question:**
After performing the update, how do you confirm that Pods are running the new image?

**Answer:**
Describe a Pod and verify the image field.

**Command:**

```bash
kubectl describe pod <pod-name> | grep -i image
```

---

## 🟡 L2 – Mid-Level (Operational Knowledge)

### Scenario 4: Wrong container name used during update

**Question:**
The update failed with the error `unable to find container named nginx`. What was the root cause?

**Answer:**
The actual container name in the Deployment was **`nginx-container`**, not `nginx`.

**Explanation:**

* Container name and image name are different entities
* The correct container name must be identified from the Deployment spec

**Command:**

```bash
kubectl describe deployment nginx-deployment
```

---

### Scenario 5: Tracking rollout progress

**Question:**
How do you monitor whether a rolling update has completed successfully?

**Answer:**
By checking the rollout status of the Deployment.

**Command:**

```bash
kubectl rollout status deployment/nginx-deployment
```

**Explanation:**

* Confirms that new Pods are created and old Pods are terminated successfully

---

### Scenario 6: Pods have different ages after update

**Question:**
Why do Pods show slightly different ages after a rolling update?

**Answer:**
Because Pods are replaced **incrementally**, not simultaneously.

**Explanation:**

* Controlled by the rolling update strategy
* Ensures service availability during updates

---

## 🔵 L3 – Senior Level (Troubleshooting & Stability)

### Scenario 7: Rollout succeeds but application is unstable

**Question:**
The rolling update completed, but the application behaves unpredictably. What should you investigate first?

**Answer:**
Check **readiness probes, liveness probes, and startup behavior**.

**Explanation:**

* Traffic is routed only to Ready Pods
* Incorrect probe configuration can cause restarts or traffic drops

---

### Scenario 8: Deployment rollout stuck

**Question:**
The rollout never completes. What are the most common causes?

**Answer:**

* Readiness probe failures
* CrashLoopBackOff in new Pods
* Insufficient node resources

**Command:**

```bash
kubectl describe pod <pod-name>
```

---

### Scenario 9: QoS Class is BestEffort

**Question:**
Why did all nginx Pods show `QoS Class: BestEffort` after deployment?

**Answer:**
Because no CPU or memory requests or limits were defined.

**Explanation:**

* BestEffort Pods have no guaranteed resources
* They are evicted first during node pressure

---

## 🔴 L4 – Architect Level (Design & Strategy)

### Scenario 10: Designing zero-downtime rollouts at scale

**Question:**
How do you design a zero-downtime rollout strategy for large-scale production workloads?

**Answer:**
Combine:

* Rolling updates
* Readiness probes
* Proper tuning of `maxSurge` and `maxUnavailable`

**Explanation:**
This ensures new Pods are ready before old ones are removed.

---

### Scenario 11: Fast recovery from bad releases

**Question:**
If the new nginx version causes issues, how do you revert safely?

**Answer:**
Perform a Deployment rollback.

**Command:**

```bash
kubectl rollout undo deployment/nginx-deployment
```

**Explanation:**

* Kubernetes reverts to the previous ReplicaSet
* Rollback is fast and low risk

---

### Scenario 12: Production hardening recommendations

**Question:**
What additional improvements would you recommend beyond this task?

**Answer:**

* Define CPU and memory requests/limits
* Add liveness and readiness probes
* Use CI/CD pipelines for image promotion
* Separate critical workloads using node affinity

**Explanation:**
These practices improve reliability, scalability, and fault tolerance.

---

## 🔑 Key Interview Takeaways

* Rolling updates are managed by Deployments
* Container name is mandatory for image updates
* Rollout status confirms deployment health
* BestEffort QoS indicates missing resource guarantees
* Rollback is a first-class Kubernetes feature

---

**Document Purpose:** Kubernetes interview preparation, real-task-based learning, and production readiness reference

### 🔑 Interview Summary Table
```
Topic	Key Insight
Rolling update	Handled by Deployment
Container name	Mandatory for set image
Zero downtime	Gradual Pod replacement
Validation	Rollout status + Pod describe
QoS	BestEffort without limits
Rollback	One-command recovery

Final Interview One-Liner

“I performed a rolling update on an nginx Deployment by identifying the correct container name, updating the image safely, monitoring rollout status, and validating that all Pods were running the new version without downtime.”
```
