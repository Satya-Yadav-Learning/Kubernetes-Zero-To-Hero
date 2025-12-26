# AWS Scenario-Based Interview Questions & Answers

## Mapping Kubernetes Shared Volume (emptyDir) Task to AWS Scenarios

---

## Scenario Context (Common)

In Kubernetes, two containers inside the same Pod share an `emptyDir` volume. When one container writes a file, the second container can immediately access it because both mount the same volume. This is a classic **sidecar/shared-storage pattern**.

In AWS, similar patterns appear when multiple processes, containers, or instances need **shared storage or data exchange**.

---

## L1 – Junior Level (Fundamentals)

### Q1. How does the Kubernetes shared volume concept map to AWS?

**Answer:**
In AWS, Kubernetes `emptyDir` is conceptually similar to:

* **EC2 instance local storage** shared by multiple processes
* **ECS containers sharing a task-level volume**

Both allow data sharing without network calls.

---

### Q2. What AWS service is closest to Kubernetes `emptyDir`?

**Answer:**

* **ECS Task ephemeral storage**
* **EC2 instance ephemeral (instance store) volume**

These storages:

* Exist only during runtime
* Are deleted when the task/instance stops

---

## L2 – Mid-Level (Operational)

### Q3. How would you implement this pattern in AWS ECS?

**Answer:**
In **ECS (EC2 or Fargate)**:

* Define a **task-level volume**
* Mount it into multiple containers

When Container A writes data, Container B reads it instantly—just like Kubernetes emptyDir.

---

### Q4. How would you verify data sharing in AWS?

**Answer:**

* Exec into container A and create a file
* Exec into container B and verify file presence

This confirms shared storage behavior.

---

## L3 – Senior Level (Design & Reliability)

### Q5. When would `emptyDir` / ephemeral storage be a bad choice?

**Answer:**

* When data must survive pod/task restarts
* When scaling across nodes or AZs
* When durability is required

AWS alternatives:

* **EFS** (shared, persistent)
* **EBS** (single-instance persistence)
* **S3** (object-level persistence)

---

### Q6. What AWS storage replaces Kubernetes persistent volumes?

**Answer:**

| Kubernetes       | AWS Equivalent        |
| ---------------- | --------------------- |
| emptyDir         | ECS ephemeral storage |
| hostPath         | EC2 instance disk     |
| PersistentVolume | EBS / EFS             |
| ReadWriteMany    | EFS                   |

---

## L4 – Architect Level (System Design)

### Q7. Design a highly available AWS system similar to this Kubernetes setup.

**Answer:**
Architecture:

* ECS or EKS cluster across multiple AZs
* Application container + sidecar container
* Shared storage via **EFS**
* Load balancer (ALB)

Benefits:

* Survives container restarts
* Supports horizontal scaling
* Centralized storage

---

### Q8. When would you use a sidecar pattern in AWS?

**Answer:**
Use sidecars for:

* Log shipping (CloudWatch agent)
* Security scanning
* Configuration reloaders
* Data preprocessing

AWS Services:

* ECS sidecar containers
* EKS sidecar pods

---

## Interview Summary (One-Liner Ready)

* Kubernetes `emptyDir` ≈ ECS task ephemeral storage
* Shared volumes enable sidecar communication
* Not suitable for persistence
* Use EFS/EBS/S3 for durable workloads
* Sidecar pattern exists across Kubernetes, ECS, and EC2

---

## Real Interview Tip

If asked:

> "File created in one container appears in another—how?"

Answer:

> "Because both containers mount the same shared volume at different paths, backed by the same underlying storage."

---

**Status:** Interview-ready
**Use Case:** Kubernetes ↔ AWS cross-platform comparison

