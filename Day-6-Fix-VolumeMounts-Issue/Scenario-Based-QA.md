# Kubernetes Scenario-Based Interview Questions & Answers

**Derived from `nginx-phpfpm` Pod Issue (L1 → L4)**

---

## Scenario Context (Common for All Levels)

A Kubernetes pod named **`nginx-phpfpm`** contains two containers:

* **Nginx** (web server)
* **PHP-FPM** (PHP execution engine)

Both containers share an **EmptyDir volume**.

### Observed Problem

* Website shows **"File not found"**
* `index.php` exists in the pod
* ConfigMap used: **`nginx-config`**

---

## 🔹 L1 – Junior Engineer (Fundamentals & Commands)

### Q1. The pod is running (2/2) but the website shows "File not found". What is the first thing you check?

**Answer:**

```bash
kubectl describe pod nginx-phpfpm
```

**Explanation:**

* Pod running status only indicates containers are up.
* `kubectl describe pod` reveals:

  * Volume mounts
  * Container names
  * Filesystem paths
* These details are essential for debugging application-level issues.

---

### Q2. Why does `kubectl cp` sometimes fail with "No such file or directory"?

**Answer:**
Because:

* `kubectl cp` defaults to the **first container** in the pod.
* The destination path may not exist in that container.

**Fix:**

```bash
kubectl cp index.php nginx-phpfpm:/path -c nginx-container
```

---

### Q3. Why must we specify `-c nginx-container`?

**Answer:**

* The pod contains multiple containers.
* Without `-c`, Kubernetes copies the file into the wrong container (`php-fpm-container`).

---

### Q4. What does an `EmptyDir` volume do?

**Answer:**

* Creates a temporary directory
* Shared between containers in the same pod
* Exists only for the lifetime of the pod

---

## 🔹 L2 – Mid-Level Engineer (Debugging & Root Cause)

### Q5. The file exists in the Nginx container but PHP still shows "File not found". Why?

**Answer:**
Because Nginx and PHP-FPM mounted the shared volume at **different paths**.

**Example:**

```text
Nginx    → /usr/share/nginx/html
PHP-FPM  → /var/www/html
```

* Nginx passes an absolute path to PHP-FPM.
* PHP-FPM cannot find that path inside its container.

---

### Q6. Which Nginx directive caused this failure?

**Answer:**

```nginx
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
```

**Explanation:**

* This sends an **absolute file path** to PHP-FPM.
* If mount paths differ, PHP-FPM cannot resolve the script.

---

### Q7. Why didn’t changing only the ConfigMap fix the issue?

**Answer:**
Because:

* The **pod specification** controls volume mount paths.
* ConfigMaps only modify application configuration, not filesystem layout.

---

### Q8. How did you confirm the real root cause?

**Answer:**
By comparing:

```bash
kubectl describe pod nginx-phpfpm
kubectl get configmap nginx-config -o yaml
```

This comparison exposed a **mountPath mismatch**.

---

## 🔹 L3 – Senior Engineer (Design & Prevention)

### Q9. What is the correct design for shared volumes in sidecar pods?

**Answer:**

> Same volume + same mountPath across all containers

**Example:**

```text
nginx-container     → /var/www/html
php-fpm-container   → /var/www/html
```

---

### Q10. Why is deleting and recreating the pod required?

**Answer:**

* Volume mounts are defined at pod creation time.
* Kubernetes does not update mount paths on running pods.
* Pod recreation is mandatory for structural fixes.

---

### Q11. What prechecks should be done before deleting a pod in production?

**Answer:**

```bash
kubectl get pod nginx-phpfpm -o jsonpath='{.spec.containers[*].image}'
kubectl get pod nginx-phpfpm -o yaml > backup.yaml
```

**Purpose:**

* Capture container images
* Preserve configuration
* Enable rollback

---

### Q12. How would you prevent this issue from reaching production?

**Answer:**

* Enforce volume mount standards
* Mandatory code review for pod YAMLs
* Integration tests validating PHP execution
* Use Helm/Kustomize with predefined mount paths

---

## 🔹 L4 – Architect Level (System Design & Standards)

### Q13. Why is this a design flaw, not an operational mistake?

**Answer:**
Because:

* Commands were correct
* ConfigMap logic was correct
* Failure resulted from filesystem abstraction mismatch

This is a **system integration issue**.

---

### Q14. What architectural pattern was used here?

**Answer:**
**Sidecar Pattern**

* Nginx → request handler
* PHP-FPM → execution engine
* Shared storage → data plane

---

### Q15. What design principles apply to this issue?

**Answer:**

* Single Source of Truth (same path everywhere)
* Principle of Least Surprise
* Contract consistency between components

---

### Q16. How would you redesign this for scale?

**Answer Options:**

* Use a shared PVC instead of `EmptyDir`
* Move PHP-FPM to a separate deployment and service
* Use read-only mounts for Nginx
* Introduce CI validation for pod specifications

---

### Q17. What is the architectural lesson learned?

**Answer:**

> In containerized systems, filesystem paths are part of the API contract.
> If that contract is inconsistent, applications fail silently but correctly.

---

## 🔹 One-Line Interview Summary (Power Answer)

> The issue occurred because the same shared volume was mounted at different absolute paths in Nginx and PHP-FPM containers, breaking FastCGI script resolution. Aligning mount paths and recreating the pod resolved the issue.

