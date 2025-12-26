# TaskAndResolution.md

## Task Title

**Shared Volume Between Containers in a Single Kubernetes Pod**

---

## Objective

Create a Kubernetes Pod with **two containers** that share a common volume using `emptyDir`.
Verify that a file created in one container is **immediately accessible** in the second container through the shared volume.

---

## Pod Details

* **Pod Name:** `volume-share-nautilus`
* **Volume Type:** `emptyDir`
* **Volume Name:** `volume-share`

### Container Configuration

| Container Name              | Image         | Mount Path     |
| --------------------------- | ------------- | -------------- |
| volume-container-nautilus-1 | fedora:latest | `/tmp/beta`    |
| volume-container-nautilus-2 | fedora:latest | `/tmp/cluster` |

---

## Pod YAML (Final)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-nautilus
spec:
  containers:
  - name: volume-container-nautilus-1
    image: fedora:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/beta

  - name: volume-container-nautilus-2
    image: fedora:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/cluster

  volumes:
  - name: volume-share
    emptyDir: {}
```

---

## Commands Executed

### 1. Apply the Pod

```bash
kubectl apply -f volume-share-nautilus.yaml
```

---

### 2. Verify Pod Status

```bash
kubectl get pod volume-share-nautilus
```

Expected:

```
STATUS: Running
READY: 2/2
```

---

### 3. Exec into First Container and Create File

```bash
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- bash
```

Inside container:

```bash
echo "Shared volume test" > /tmp/beta/beta.txt
exit
```

---

### 4. Verify File in Second Container

```bash
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- bash
```

Inside container:

```bash
cat /tmp/cluster/beta.txt
```

Expected output:

```
Shared volume test
```

---

## Resolution Summary

The task was successfully completed.
A file created in the first container was **immediately visible** in the second container, confirming correct shared volume behavior.

---

## Explanation: Why Did the File Appear in the Second Container?

### Core Reason

Both containers mount the **same `emptyDir` volume**, so they operate on the **same underlying filesystem directory**, even though the mount paths differ.

---

### How Kubernetes Handles This Internally

1. **Volume Created Once Per Pod**
   Kubernetes creates a single directory on the node for the `emptyDir` volume.

2. **Same Volume Mounted into Both Containers**

   * Container 1 mounts it at `/tmp/beta`
   * Container 2 mounts it at `/tmp/cluster`

3. **Result**
   Writing a file in one container instantly makes it available in the other.

---

## Important Clarifications

* Containers do **not** copy files between each other
* Kubernetes does **not** replicate the data
* Sharing works **only because**:

  * Containers are in the **same Pod**
  * They mount the **same volume**

---

## Pod Scope Limitation

| Scope          | Can Share `emptyDir` |
| -------------- | -------------------- |
| Same Pod       | Yes                  |
| Different Pods | No                   |

`emptyDir` volumes are **Pod-scoped**, not cluster-wide.

---

## Real-World Use Case (Sidecar Pattern)

| Container | Responsibility                       |
| --------- | ------------------------------------ |
| Main App  | Writes logs to shared volume         |
| Sidecar   | Reads logs and ships them externally |

This pattern avoids API calls, network overhead, and log duplication.

---

## Interview-Ready One-Liner

> Files appear in both containers because `emptyDir` volumes are created once per Pod and shared across containers, allowing different mount paths to access the same underlying storage.

---

## Final State Confirmation

* Pod is **Running**
* Both containers are **Ready**
* File created in container 1 is accessible in container 2
* Shared volume works as expected

