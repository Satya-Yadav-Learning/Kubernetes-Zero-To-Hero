# TaskAndResolution.md

## Task Title

Shared Volume with Sidecar Pattern using emptyDir in Kubernetes

---

## Objective

Create a Kubernetes Pod with two containers that share an `emptyDir` volume. One container runs **nginx** (main application) and the other runs **ubuntu** as a **sidecar container** to continuously read nginx logs from the shared volume.

This task demonstrates:

* Shared volumes in Kubernetes
* Sidecar container pattern
* Real-time log consumption using `emptyDir`

---

## Pod Requirements

* Pod Name: `webserver`
* Volume Name: `shared-logs`
* Volume Type: `emptyDir`

### Containers

| Container Name    | Image         | Purpose                    |
| ----------------- | ------------- | -------------------------- |
| nginx-container   | nginx:latest  | Web server generating logs |
| sidecar-container | ubuntu:latest | Sidecar to read nginx logs |

### Volume Mount Path (Both Containers)

```
/var/log/nginx
```

---

## YAML Definition Used

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}

  containers:
  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  - name: sidecar-container
    image: ubuntu:latest
    command: ["sh", "-c", "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
```

---

## Step-by-Step Execution & Verification

### 1️⃣ Apply the Pod YAML

```bash
kubectl apply -f webserver.yaml
```

**Output:**

```
pod/webserver created
```

**Explanation:**
Creates the Pod with both containers and the shared `emptyDir` volume.

---

### 2️⃣ Verify Pod Status

```bash
kubectl get pod webserver
```

**Output:**

```
NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          19s
```

**Explanation:**
Confirms both containers are running successfully.

---

### 3️⃣ Describe Pod (Volume & Mount Verification)

```bash
kubectl describe pod webserver
```

**Relevant Output:**

```
Containers:
  nginx-container:
    Mounts:
      /var/log/nginx from shared-logs (rw)

  sidecar-container:
    Mounts:
      /var/log/nginx from shared-logs (rw)

Volumes:
  shared-logs:
    Type: EmptyDir
```

**Explanation:**
Validates that both containers mount the same `emptyDir` volume at the same path.

---

### 4️⃣ Generate Traffic (Create Logs)

```bash
kubectl exec webserver -c nginx-container -- curl localhost
```

**Output (HTML):**

```
Welcome to nginx!
```

**Explanation:**
The HTTP request generates entries in `access.log` and `error.log`.

---

### 5️⃣ Verify Sidecar is Reading Logs

```bash
kubectl logs webserver -c sidecar-container
```

**Output (Excerpt):**

```
::1 - - [26/Dec/2025:06:16:53 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.14.1"
```

**Explanation:**
Confirms:

* nginx wrote logs to `/var/log/nginx`
* sidecar container read the same logs via shared volume

---

## Key Observations

* `emptyDir` is created once per Pod
* Data written by one container is immediately visible to others
* Sidecar pattern avoids modifying the main application container
* Logs are processed independently

---

## Why Sidecar Pattern is Used

* Separation of concerns
* Cleaner application containers
* Independent lifecycle for operational tasks
* Commonly used for:

  * Logging
  * Monitoring
  * Security agents
  * File shipping

---

## Important Notes

* `emptyDir` data is deleted when Pod is deleted
* Not suitable for persistent storage
* Best for temporary, Pod-scoped data sharing

---

## Final Status

✔ Pod running successfully
✔ Shared volume verified
✔ Sidecar reading logs correctly
✔ Task completed as expected

---

## Interview-Ready Summary

> This task demonstrates how Kubernetes uses `emptyDir` volumes to enable file sharing between containers in the same Pod, commonly implemented using the sidecar pattern for log processing and observability use cases.

