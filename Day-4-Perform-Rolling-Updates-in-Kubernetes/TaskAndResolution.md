# Task and Resolution: Rolling Update of nginx Deployment in Kubernetes

---

## Task Description

The application development team introduced new changes packaged in a Docker image **`nginx:1.19`**. The requirement was to perform a **rolling update** on an existing Kubernetes Deployment named **`nginx-deployment`**, ensuring:

* Zero downtime during the update
* All Pods are updated to the new image
* All Pods are healthy and running after the update

> The `kubectl` utility is preconfigured and connected to the target Kubernetes cluster.

---

## Step 1: Verify Kubernetes Context

```bash
kubectl config get-contexts
```

**Output:**

```text
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         kind-notecloud   kind-notecloud   kind-notecloud
```

**Explanation:**

* Confirms the active Kubernetes context
* Ensures commands are executed against the correct cluster

---

## Step 2: Verify Node Status

```bash
kubectl get nodes
```

**Output:**

```text
NAME                      STATUS   ROLES           AGE   VERSION
notecloud-control-plane   Ready    control-plane   15m   v1.27.16-1+f5da3b717fc217
```

**Explanation:**

* Confirms the node is in `Ready` state
* Ensures the cluster can schedule and run Pods

---

## Step 3: Verify Existing Deployment

```bash
kubectl get deployment nginx-deployment
```

**Output:**

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           17m
```

**Explanation:**

* Confirms the Deployment exists
* All replicas are currently running and available

---

## Step 4: Identify Container Name in Deployment

```bash
kubectl describe deployment nginx-deployment
```

**Output (Relevant Section):**

```text
Containers:
  nginx-container:
    Image:  nginx:1.16
```

**Explanation:**

* `kubectl set image` requires the **exact container name**
* Container name is **`nginx-container`**, not `nginx`

---

## Step 5: Perform Rolling Update

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.19
```

**Output:**

```text
deployment.apps/nginx-deployment image updated
```

**Explanation:**

* Triggers a rolling update
* Kubernetes creates a new ReplicaSet
* Old Pods are replaced gradually

---

## Step 6: Monitor Rollout Status

```bash
kubectl rollout status deployment/nginx-deployment
```

**Output:**

```text
deployment "nginx-deployment" successfully rolled out
```

**Explanation:**

* Confirms rolling update completed successfully
* No Pod failures or rollbacks occurred

---

## Step 7: Verify Pod Status

```bash
kubectl get pods
```

**Output:**

```text
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-dc49f85cc-4xsk2   1/1     Running   0          4m
nginx-deployment-dc49f85cc-h6ws6   1/1     Running   0          4m
nginx-deployment-dc49f85cc-nbgqj   1/1     Running   0          4m
```

**Explanation:**

* All Pods are in `Running` state
* `READY 1/1` confirms container health

---

## Step 8: Verify Image Version on Pods

```bash
kubectl describe pod nginx-deployment-dc49f85cc-4xsk2 | grep -i image
```

**Output:**

```text
Image: nginx:1.19
Image ID: docker.io/library/nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
```

**Explanation:**

* Confirms the Pod is running the updated image
* Ensures old image version is no longer used

---

## Step 9: Count Running Containers

```bash
kubectl get pods -o jsonpath='{range .items[*]}{range .status.containerStatuses[*]}{.state.running}{"\n"}{end}' | wc -l
```

**Output:**

```text
3
```

**Explanation:**

* Confirms total running containers
* Matches the desired replica count

---

## Final Outcome

* Rolling update executed successfully
* Deployment updated from `nginx:1.16` to `nginx:1.19`
* All Pods running and healthy
* Zero downtime achieved

---

## Key Learnings (Interview Ready)

* Rolling updates are handled by the Deployment controller
* `kubectl set image` requires `container-name=image` format
* Readiness of Pods ensures zero downtime
* ReplicaSet hash change confirms version update

---

**Status:** Task completed and validated successfully

