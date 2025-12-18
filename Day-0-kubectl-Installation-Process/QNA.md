# QNA.md

## Scenario-Based Interview Questions & Answers

### Topic: kubectl Installation, Configuration & Kubernetes Operations

### Levels: L1 → L4 (Junior → Senior → Architect)

---

## L1 – Junior / Fresher Level

### Q1. Scenario: `kubectl` command not found

**Question:** You run `kubectl get pods` and get `command not found`. What do you do?

**Answer:**
First, I check whether `kubectl` is installed and available in the system PATH.

```bash
which kubectl
echo $PATH
```

If it is not found, I install `kubectl` and move it to `/usr/local/bin`.

**Explanation:**
Linux can only execute binaries that are present in directories listed in the PATH environment variable.

---

### Q2. Scenario: Permission denied while running kubectl

**Question:** kubectl exists but shows `permission denied` when executed.

**Answer:**

```bash
chmod +x kubectl
```

**Explanation:**
The executable permission is missing. Without it, the OS cannot run the binary.

---

### Q3. Scenario: kubectl installed but cannot connect to cluster

**Question:** `kubectl version --client` works but `kubectl get pods` fails.

**Answer:**

```bash
kubectl config get-contexts
kubectl config current-context
```

**Explanation:**
This indicates that kubectl is installed, but kubeconfig is either missing or pointing to the wrong cluster.

---

### Q4. Scenario: Multiple clusters configured

**Question:** How do you know which cluster kubectl is currently using?

**Answer:**

```bash
kubectl config current-context
```

**Explanation:**
The current context defines the active cluster, user, and namespace.

---

## L2 – Associate / Mid-Level DevOps

### Q5. Scenario: Client and server version mismatch

**Question:** kubectl shows a version skew warning. What does it mean?

**Answer:**
It means the kubectl client version is not compatible with the cluster version.

**Explanation:**
Kubernetes supports a version skew of ±1 minor version between client and server.

---

### Q6. Scenario: User gets `Forbidden` error

**Question:** kubectl is installed but access is denied.

**Answer:**

```bash
kubectl auth can-i get pods
```

**Explanation:**
This is an RBAC issue. Installation does not grant permissions automatically.

---

### Q7. Scenario: Command executed on wrong cluster

**Question:** A command affected the wrong environment. How do you prevent this?

**Answer:**

```bash
kubectl config use-context dev-cluster
```

**Explanation:**
Explicit context switching prevents accidental changes in production.

---

### Q8. Scenario: kubectl works on jump host but not on laptop

**Question:** Why does kubectl fail locally?

**Answer:**
The kubeconfig is missing or not configured locally.

**Explanation:**
Each system requires its own kubeconfig configuration.

---

## L3 – Senior DevOps / SRE Level

### Q9. Scenario: kubectl hangs during production outage

**Question:** kubectl commands are not responding. What do you check?

**Answer:**

```bash
kubectl cluster-info
```

**Explanation:**
If the API server is unreachable, kubectl cannot function because it is only a client.

---

### Q10. Scenario: Securing kubectl access in production

**Question:** How do you secure kubectl usage?

**Answer:**

* Use RBAC
* Use IAM / SSO authentication
* Limit kubeconfig file permissions
* Avoid shared credentials

**Explanation:**
Anyone with kubectl access can potentially control the cluster.

---

### Q11. Scenario: Debugging application failures

**Question:** Pods are failing even though kubectl is working.

**Answer:**

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

**Explanation:**
kubectl is the primary interface for debugging runtime and configuration issues.

---

### Q12. Scenario: Different kubectl versions across team

**Question:** Why is this a problem and how do you fix it?

**Answer:**
Standardize kubectl versions using documentation or automation.

**Explanation:**
Different versions can behave differently and cause unexpected issues.

---

## L4 – Architect / Platform Engineer Level

### Q13. Scenario: Enterprise-scale kubectl access

**Question:** How do you design kubectl access for large teams?

**Answer:**

* Central identity (IAM/SSO)
* Role-based access
* Short-lived credentials
* Audit logging

**Explanation:**
This ensures scalability, security, and traceability.

---

### Q14. Scenario: kubectl usage in GitOps

**Question:** Should engineers use kubectl in production?

**Answer:**
Only for read-only operations. Changes should go through GitOps pipelines.

**Explanation:**
This maintains Git as the single source of truth.

---

### Q15. Scenario: Cluster recovery when kubectl is unavailable

**Question:** How do you recover during a SEV-0 incident?

**Answer:**

* Use GitOps rollback
* Use cloud provider tools as last resort

**Explanation:**
kubectl is not the only recovery mechanism in well-designed systems.

---

## Interview Summary

* L1 focuses on installation and basic usage
* L2 focuses on troubleshooting and correctness
* L3 focuses on reliability and security
* L4 focuses on design, governance, and scale

This document can be directly used for interview preparation and GitHub portfolios.

