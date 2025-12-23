
## Task Details

**Pod Name:** `nginx-phpfpm`
**ConfigMap Name:** `nginx-config`

**Task Requirement:**

1. Identify and fix the issue preventing the website from loading.
2. Copy `/home/thor/index.php` from the jump host into the **nginx container** inside the Nginx document root.
3. Verify that the website is accessible using the **Website** button on the top bar.

---

## Environment Overview

* Pod contains **two containers (sidecar pattern)**:

  * `nginx-container` → Web server
  * `php-fpm-container` → PHP execution engine
* Shared storage via `EmptyDir` volume
* Configuration injected via ConfigMap

---

## Pre-Checks and Initial Observations

### 1. Verify Pod Status

```bash
kubectl get pods
```

**Output:**

```text
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          3m
```

**Explanation:**

* Pod is running and both containers are healthy.
* Issue is not related to pod crash or image pull failure.

---

### 2. Describe the Pod (Critical Step)

```bash
kubectl describe pod nginx-phpfpm
```

**Key Output (Relevant Sections):**

```text
Containers:
  php-fpm-container:
    Image: php:7.2-fpm-alpine
    Mounts:
      /var/www/html from shared-files (rw)

  nginx-container:
    Image: nginx:latest
    Mounts:
      /usr/share/nginx/html from shared-files (rw)
```

**Finding:**

* The same `EmptyDir` volume (`shared-files`) is mounted at **different paths** in the two containers.

---

### 3. Inspect ConfigMap

```bash
kubectl get configmap nginx-config -o yaml
```

**Output:**

```yaml
root /var/www/html;
```

**Finding:**

* Nginx is configured to serve files from `/var/www/html`.
* However, Nginx container volume was mounted at `/usr/share/nginx/html`.

---

## Root Cause Analysis (RCA)

### Problem Identified

* Nginx and PHP-FPM containers were sharing the same volume **but mounted at different absolute paths**.
* Nginx passed an absolute file path to PHP-FPM using `SCRIPT_FILENAME`.
* PHP-FPM could not find the file at that path inside its container.

### Resulting Error

* Website showed **"File not found"** even though the file existed.

### Key Insight

> In multi-container pods, when absolute paths are exchanged (as with FastCGI), the shared volume **must be mounted at the same path in all containers**.

---

## Pre-Resolution Image Capture (Mandatory Pre-Check)

Before making any destructive changes (such as deleting the pod), image details were captured to ensure traceability and rollback capability.

### Capture Container Images Used by the Pod

```bash
kubectl get pod nginx-phpfpm -o jsonpath='{range .spec.containers[*]}{.name}{" -> "}{.image}{"
"}{end}'
```

**Output:**

```text
php-fpm-container -> php:7.2-fpm-alpine
nginx-container -> nginx:latest
```

**Explanation:**

* Lists all containers inside the pod with their exact image names and tags.
* This is critical for audit, rollback, and reproducibility.

---

### Backup Pod Definition Before Deletion

```bash
kubectl get pod nginx-phpfpm -o yaml > nginx-phpfpm-before-delete.yaml
```

**Explanation:**

* Exports the full pod specification to a local YAML file.
* Ensures the pod can be recreated exactly as it existed before deletion.

---

## Resolution Steps

### Step 1: Fix Pod Volume Mount Paths

Edit the saved pod manifest:

```bash
vi nginx-phpfpm-before-delete.yaml
```

**Correct Configuration:**

Both containers must mount the shared volume at `/var/www/html`.

```yaml
volumeMounts:
- name: shared-files
  mountPath: /var/www/html
```

**Explanation:**

* Ensures Nginx and PHP-FPM resolve file paths identically.

---

### Step 2: Ensure ConfigMap Uses the Same Document Root

```bash
kubectl edit configmap nginx-config
```

**Correct Directive:**

```nginx
root /var/www/html;
```

---

### Step 3: Recreate the Pod

```bash
kubectl delete pod nginx-phpfpm
kubectl apply -f nginx-phpfpm-before-delete.yaml
```

**Verification:**

```bash
kubectl get pods
```

```text
nginx-phpfpm   2/2   Running
```

---

### Step 4: Copy `index.php` into the Nginx Container

```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

**Explanation:**

* `-c nginx-container` ensures the correct container is targeted.
* `/var/www/html` matches both the volume mount and Nginx root.

---

### Step 5: Verify File Presence (Optional Validation)

```bash
kubectl exec -it nginx-phpfpm -c nginx-container -- ls -l /var/www/html
```

**Expected Output:**

```text
index.php
```

---

## Final Validation

* PHP page loads successfully.

---

## Final Outcome

* Pod YAML corrected
* ConfigMap aligned with volume mounts
* PHP-FPM and Nginx paths unified
* Website accessible successfully

---

## One-Line Summary (Interview Ready)

> The issue was caused by mounting the same shared volume at different paths in Nginx and PHP-FPM containers, which broke FastCGI script resolution. Aligning mount paths and recreating the pod resolved the issue.

---

## Key Learnings

* Always validate **volume mount paths** in multi-container pods.
* Image defaults are irrelevant once paths are overridden in pod specs.
* `kubectl describe pod` is the most reliable source of truth for debugging runtime issues.

