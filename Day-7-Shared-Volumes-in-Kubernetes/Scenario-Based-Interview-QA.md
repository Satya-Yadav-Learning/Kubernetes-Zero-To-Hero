# Scenario Based Interview Questions & Answers

## Kubernetes Shared Volume (emptyDir) – L1 to L4

---

## L1 – Junior DevOps / Kubernetes Fundamentals

### Q1. What was the goal of this task?

**Answer:**
The goal was to demonstrate how multiple containers inside a single Kubernetes Pod can share data using a shared volume (`emptyDir`). A file created in one container should be immediately accessible from another container.

---

### Q2. What is an `emptyDir` volume?

**Answer:**
An `emptyDir` is a temporary Kubernetes volume that:

* Is created when the Pod starts
* Exists as long as the Pod is running
* Is deleted when the Pod is removed

It is commonly used for temporary storage and inter-container communication.

---

### Q3. Why were two containers created in the same Pod?

**Answer:**
Containers in the same Pod:

* Share the same network namespace
* Can mount the same volume

This makes them ideal for tightly coupled workloads that need fast local data sharing.

---

### Q4. Why did the file appear in the second container automatically?

**Answer:**
Both containers mounted the same `emptyDir` volume (`volume-share`). Even though the mount paths were different (`/tmp/beta` and `/tmp/cluster`), they pointed to the same underlying storage.

---

### Q5. How did you verify the shared volume worked?

**Answer:**

1. Created a file `beta.txt` inside container-1
2. Checked for the same file in container-2
3. The file was visible without copying, confirming shared volume functionality

---

## L2 – Mid-Level DevOps / Troubleshooting & Internals

### Q6. Why was `sleep 3600` used in both containers?

**Answer:**
Containers stop when their main process exits. The `sleep` command keeps the containers running so that we can exec into them for testing.

---

### Q7. What happens to `emptyDir` data if the Pod restarts?

**Answer:**
All data inside `emptyDir` is lost when the Pod is deleted or restarted. The volume is recreated empty.

---

### Q8. Can containers in different Pods share an `emptyDir` volume?

**Answer:**
No. `emptyDir` is Pod-scoped. Only containers within the same Pod can share it.

---

### Q9. Why was `kubectl` not available inside the container?

**Answer:**
Application containers are minimal and usually do not include administrative tools like `kubectl`. Cluster operations should be performed from a jump host or admin workstation.

---

## L3 – Senior DevOps / Real-World Usage

### Q10. Is this an example of the sidecar pattern?

**Answer:**
Yes. This setup demonstrates the foundation of the sidecar pattern, where multiple containers share volumes and work together inside a Pod.

---

### Q11. Provide a real-world use case for this pattern.

**Answer:**
Common examples include:

* Application container writing logs
* Sidecar container collecting and shipping logs
* Both containers sharing a volume

---

### Q12. Why not use `kubectl cp` instead of a shared volume?

**Answer:**
`kubectl cp` is manual and not real-time. Shared volumes enable automatic, continuous data sharing without manual intervention.

---

### Q13. How would you make this data persistent?

**Answer:**
Use a PersistentVolume (PV) and PersistentVolumeClaim (PVC) instead of `emptyDir`.

---

## L4 – Architect / System Design Level

### Q14. When should `emptyDir` not be used?

**Answer:**
Avoid `emptyDir` when:

* Data must survive Pod restarts
* Data is business-critical
* Multiple Pods need access

---

### Q15. Compare Sidecar Containers vs Init Containers.

| Feature   | Sidecar              | Init Container         |
| --------- | -------------------- | ---------------------- |
| Execution | Runs with app        | Runs before app        |
| Lifetime  | Entire Pod lifecycle | Ends before app starts |
| Use Case  | Logging, proxy       | Setup, migration       |

---

### Q16. What are the risks of shared volumes?

**Answer:**

* Concurrent write conflicts
* Accidental deletion
* Lack of isolation

Mitigation includes read-only mounts and clear ownership rules.

---

## Interview Summary

> This task demonstrates how Kubernetes uses shared volumes and sidecar-style containers to enable real-time data sharing within a Pod using `emptyDir` volumes.

