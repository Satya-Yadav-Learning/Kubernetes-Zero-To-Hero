# Scenario-Based Interview Questions & Answers

## Kubernetes Deployment Rollback – `nginx-deployment` (notecloud Cluster)

---

This document contains **scenario-based interview questions and answers** derived directly from a **real rollback operation** performed on a Kubernetes Deployment named **`nginx-deployment`** in the **notecloud** cluster.

The questions are structured from **L1 (Junior)** to **L4 (Architect)** and focus on **rollback decision-making, validation, troubleshooting, and production-grade reasoning**.

---

## 🟢 L1 – Junior Level (Fundamentals)

### Scenario 1: Bug reported after deployment

**Question:**
A new version of an application was deployed, and customers reported a bug. What is the fastest and safest way to revert to the previous working version in Kubernetes?

**Answer:**
Use a **Deployment rollback** with `kubectl rollout undo`.

**Explanation:**

* Kubernetes Deployments maintain revision history
* Rollback switches traffic back to the previous ReplicaSet
* No YAML changes or redeployment are required

**Command:**

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

### Scenario 2: Verifying rollback feasibility

**Question:**
Before performing a rollback, how do you confirm that a previous version exists?

**Answer:**
By checking the rollout history of the Deployment.

**Explanation:**

* Each revision represents a ReplicaSet
* Multiple revisions indicate rollback capability

**Command:**

```bash
kubectl rollout history deployment/nginx-deployment
```

---

## 🟡 L2 – Mid-Level (Operations & Validation)

### Scenario 3: Ensuring the correct cluster

**Question:**
Why is it critical to verify the Kubernetes context before executing a rollback?

**Answer:**
To ensure the rollback is executed on the intended cluster and environment.

**Explanation:**

* Prevents accidental rollbacks in the wrong environment (e.g., production vs staging)
* This is a mandatory safety check in real operations

**Commands:**

```bash
kubectl config current-context
kubectl config get-contexts
kubectl cluster-info
```

---

### Scenario 4: Pod names change after rollback

**Question:**
After rollback, Pods have new names with a different hash. Is this expected behavior?

**Answer:**
Yes, this is expected.

**Explanation:**

* Rollback reactivates the previous ReplicaSet
* Kubernetes creates new Pods from that ReplicaSet
* Old Pods are terminated gracefully

---

### Scenario 5: Validating rollback success

**Question:**
What checks confirm that the rollback completed successfully?

**Answer:**

* Pods are in `Running` state
* Deployment rollout status shows success

**Commands:**

```bash
kubectl get pods
kubectl rollout status deployment/nginx-deployment
```

---

## 🔵 L3 – Senior Level (Troubleshooting & Reliability)

### Scenario 6: Issue persists after rollback

**Question:**
Rollback completed successfully, but users still experience issues. What should you investigate next?

**Answer:**
Verify the application configuration, image version, and external dependencies.

**Explanation:**

* Not all issues are version-related
* ConfigMaps, Secrets, or backend services may be the root cause

---

### Scenario 7: Node health verification

**Question:**
Why did you verify node status before executing the rollback?

[O**Answer:**
Because rollback requires scheduling new Pods on healthy nodes.

**Explanation:**

* If nodes are `NotReady`, rollback Pods may fail
* Node health directly affects rollout success

**Command:**

```bash
kubectl get nodes -o wide
```

---

### Scenario 8: Namespace awareness

**Question:**
Why is it important to confirm the namespace before rollback?

**Answer:**
Because `kubectl` defaults to the `default` namespace.

**Explanation:**

* Wrong namespace may lead to resource not found errors
* Or cause rollback of the wrong application

**Command:**

```bash
kubectl get deployments -A
```

---

## 🔴 L4 – Architect Level (Design & Strategy)

### Scenario 9: Rollback vs redeploy

**Question:**
Why is `kubectl rollout undo` preferred over redeploying the old image manually?

**Answer:**
Rollback is safer, faster, and less error-prone.

**Explanation:**

* Uses previously tested ReplicaSets
* Preserves metadata, labels, and selectors
* Minimizes human error during incidents

---

### Scenario 10: Zero-downtime rollback

**Question:**
How does Kubernetes ensure zero downtime during a rollback?

**Answer:**
By using the same rolling update mechanism in reverse.

**Explanation:**

* New Pods from the previous ReplicaSet become Ready first
* Old Pods are terminated only after readiness is confirmed
* Services continue routing traffic seamlessly

---

### Scenario 11: Production hardening for rollbacks

**Question:**
What best practices improve rollback reliability in production?

**Answer:**

* Configure readiness and liveness probes
* Define CPU and memory requests/limits
* Use `--record=true` for audit trails
* Integrate rollback into CI/CD pipelines

**Explanation:**
These practices reduce risk and accelerate incident recovery.

---

## 🔑 Interview Summary (One-Liner)

> “I validated the Kubernetes context, cluster, nodes, and deployment history before performing a rollback using `kubectl rollout undo`, monitored rollout status, and confirmed that the previous stable version was restored successfully.”

---

## Final Takeaways

* Rollbacks are **Deployment-native** and safe
* ReplicaSets enable fast reversions
* Validation before and after rollback is mandatory
* This approach is production-ready and audit-friendly

---

**Document Purpose:** Kubernetes rollback interview preparation, real-task-based learning, and production readiness reference

