# AWS Scenario-Based Interview Questions & Answers

**Mapped from Kubernetes Nginx + PHP-FPM Volume Path Issue**

---

## Reference Scenario (Source Issue)

* Kubernetes pod: `nginx-phpfpm`
* Containers: Nginx + PHP-FPM (sidecar pattern)
* Shared volume: `EmptyDir`
* Issue: Website showed **"File not found"**
* Root Cause: **Same shared volume mounted at different absolute paths**

This document maps the same failure pattern to **real AWS production scenarios**.

---

## L1 – Junior Level (AWS Fundamentals)

### Q1. An application behind an AWS ALB returns 404, but the EC2 instance is healthy. What do you check first?

**Answer:**

* Verify the web server document root configuration.
* Confirm the requested file exists at that path.

**AWS Mapping:**

* ALB health checks validate port and HTTP response only.
* They do not validate filesystem paths or application logic.

---

### Q2. Why can an ALB target group show "healthy" while users see errors?

**Answer:**

* ALB health checks are shallow.
* Application-level misconfigurations (paths, permissions) are not detected.

---

### Q3. In ECS, why might a task be running but the application still fail?

**Answer:**

* Containers can start successfully even if volume mount paths or environment variables are incorrect.
* Similar to Kubernetes pods showing `Running` while the app is broken.

---

## L2 – Mid-Level (Debugging & RCA)

### Q4. After migrating from EC2 to ECS, PHP applications return "File not found". Why?

**Answer:**

* Container filesystem paths differ from EC2 paths.
* Application configs still reference old EC2 directories.

---

### Q5. What AWS services behave like Kubernetes `EmptyDir`?

**Answer:**

* ECS ephemeral storage
* EC2 instance store
* AWS Lambda `/tmp`

All are:

* Temporary
* Lifecycle-bound
* Path-sensitive

---

### Q6. Why doesn’t changing only the web server config resolve such issues?

**Answer:**

* The root cause is infrastructure design (mount paths), not application config.
* Similar to fixing Nginx config without fixing volume mounts.

---

## L3 – Senior Level (Design & Prevention)

### Q7. Real AWS scenario: ECS + EFS based PHP app outage. Likely cause?

**Answer:**

* EFS mounted at a different path than expected by PHP-FPM.
* Application fails despite ECS service being healthy.

---

### Q8. Which AWS components are commonly involved in this failure?

**Answer:**

* ECS task definitions
* EFS volume mounts
* Container image defaults
* Environment-specific configs

---

### Q9. How would you prevent this issue in AWS?

**Answer:**

* Enforce standard mount paths via Terraform modules
* Use golden AMIs or standardized ECS task definitions
* Add CI validation for mount paths

---

## L4 – Architect Level (System Design & Governance)

### Q10. Why is this a design flaw rather than an operational error?

**Answer:**

* Infrastructure was healthy
* Commands were correct
* Failure occurred at the **interface contract level** between components

---

### Q11. Which AWS Well-Architected pillars are impacted?

**Answer:**

* Reliability – silent failures
* Operational Excellence – lack of validation
* Security – inconsistent paths often lead to permission drift

---

### Q12. How would you prevent this class of issue across multiple AWS accounts?

**Answer:**

* Golden AMIs
* ECS task definition templates
* Policy-as-code (OPA, Sentinel)
* CI/CD checks validating filesystem contracts

---

## Real AWS Production Incident RCAs

### Incident 1: ECS + EFS PHP Outage

**Cause:**

* EFS mounted at `/mnt/efs`
* App expected `/var/www/html`

**Impact:**

* 100% PHP endpoint failure

**Fix:**

* Align mount paths across all containers

---

### Incident 2: EC2 Auto Scaling Deployment Failure

**Cause:**

* New AMI used different document root

**Impact:**

* New instances served empty content

**Lesson:**

* Filesystem paths are part of the API contract

---

### Incident 3: Blue/Green ECS Deployment Failure

**Cause:**

* New task revision changed mount path

**Impact:**

* Partial production outage during deployment

**Prevention:**

* Schema and contract validation in CI

---

## One-Line Interview Power Answer

> The outage occurred because shared storage was mounted at different absolute paths across application components, breaking the filesystem contract—a common architectural failure during ECS, EKS, and EC2 migrations.

---

## Universal Takeaway

> Filesystem paths are not implementation details; they are **integration contracts** in distributed systems.

