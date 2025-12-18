# AWS-Incident-Scenarios.md

## Real-World AWS Incident Scenarios (Mapped to Kubernetes & kubectl)

This document captures **real AWS production incident scenarios** commonly faced in **EKS-based environments**, mapped to Kubernetes and `kubectl` operations. These scenarios are written in an **on-call / interview-ready** format.

---

## Incident 1: kubectl Works but Cluster Is Unreachable (SEV-1)

### Scenario

* `kubectl version --client` works
* `kubectl get pods` hangs or times out

### AWS Root Cause

* EKS API server endpoint not reachable
* VPN / Bastion / jump host network issue
* Private endpoint without proper routing

### Diagnosis Steps

```bash
kubectl cluster-info
kubectl config current-context
```

### AWS Checks

* VPC route tables
* Security Groups
* EKS endpoint access (public/private)

### Resolution

* Restore network connectivity
* Update routing or endpoint access

---

## Incident 2: Access Denied / Forbidden Error (SEV-2)

### Scenario

* kubectl installed and configured
* Error: `Forbidden` or authentication failure

### AWS Root Cause

* IAM role/user not mapped in `aws-auth` ConfigMap
* Missing Kubernetes RBAC permissions

### Diagnosis

```bash
kubectl auth can-i get pods
```

### Resolution

* Update `aws-auth` ConfigMap
* Grant appropriate RBAC roles

---

## Incident 3: Command Executed on Wrong Cluster (SEV-0 Near Miss)

### Scenario

* Changes intended for dev affected production

### Root Cause

* Multiple kubeconfig contexts
* No context validation

### Prevention

```bash
kubectl config get-contexts
kubectl config use-context dev
```

### Best Practice

* Separate AWS accounts per environment
* Read-only access for production

---

## Incident 4: Pods Stuck in Pending State (SEV-1)

### Scenario

* Deployment created
* Pods not scheduled

### AWS Root Cause

* EC2 node group capacity exhausted
* AZ capacity issues

### Diagnosis

```bash
kubectl describe pod <pod-name>
kubectl get nodes
```

### Resolution

* Scale node group
* Enable Cluster Autoscaler

---

## Incident 5: kubectl Works on Jump Host but Not Locally (SEV-3)

### Scenario

* kubectl commands fail on laptop

### Root Cause

* Missing kubeconfig locally
* Expired AWS credentials

### Fix

```bash
aws eks update-kubeconfig --region us-east-1 --name <cluster-name>
```

---

## Incident 6: kubectl Unresponsive During Outage (SEV-0)

### Scenario

* ALB returning 5xx
* kubectl commands not responding

### Root Cause

* EKS control plane degradation
* Network partition

### Recovery

* GitOps rollback
* AWS Console (last resort)

---

## Incident 7: Pod Crashes After Deployment (SEV-1)

### Scenario

* Pods restart frequently

### Root Cause

* Image tag `latest` used
* Breaking change introduced

### Diagnosis

```bash
kubectl describe pod <pod>
kubectl logs <pod>
```

### Resolution

* Pin image versions
* Roll back Deployment

---

## Incident 8: Sudden AWS Cost Spike (Cost Incident)

### Scenario

* Unexpected increase in AWS bill

### Root Cause

* No resource limits
* HPA and Cluster Autoscaler runaway

### Diagnosis

```bash
kubectl describe hpa
kubectl top pods
```

### Resolution

* Set resource limits
* Enforce scaling boundaries

---

## Incident 9: Unauthorized Resource Creation (Security – SEV-0)

### Scenario

* Unknown workloads appear

### Root Cause

* Over-permissive IAM roles
* Shared kubeconfig files

### Resolution

* Rotate credentials
* Enforce IAM + RBAC
* Enable CloudTrail and audit logs

---

## Incident 10: No Audit Trail for Production Changes

### Scenario

* Manual kubectl changes
* No Git history

### Root Cause

* No GitOps enforcement

### Resolution

* Implement ArgoCD / Flux
* Restrict kubectl write access in prod

---

## High-Value AWS Mapping Table

| Kubernetes Concept | AWS Component     |
| ------------------ | ----------------- |
| kubectl            | Client Tool       |
| API Server         | EKS Control Plane |
| Node               | EC2               |
| Authentication     | IAM               |
| Authorization      | RBAC              |
| Service            | ALB / NLB         |
| Logs               | CloudWatch        |
| GitOps             | CI/CD + ArgoCD    |

---

## Interview Key Takeaway

> "Most AWS EKS incidents are IAM, networking, or capacity issues. kubectl is only the control and visibility layer."

---

## Conclusion

These scenarios reflect **real AWS production incidents** and demonstrate how Kubernetes knowledge, AWS services, and operational discipline intersect in real-world environments.
















































































































































































