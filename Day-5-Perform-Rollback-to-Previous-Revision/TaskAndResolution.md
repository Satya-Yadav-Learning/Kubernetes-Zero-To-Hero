
## Task

New release of an application was deployed. A customer reported a bug in this release, and DevOps team decided to **roll back the Kubernetes Deployment to the previous stable revision**.

* **Deployment name:** nginx-deployment
* **Action required:** Roll back to the previous revision
* **Environment:** Kubernetes cluster accessed via `kubectl`

---

## Pre-Checks (Executed Exactly as Run)

### 1. Check Deployment Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

**Output:**

```text
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine-perl --kubeconfig=/root/.kube/config --record=true
```

**Explanation:**

* Two revisions exist for the Deployment
* Revision **2** is the latest (buggy) release
* Revision **1** is the previous stable version
* Confirms that a rollback is possible

---

### 2. Verify Pods Before Rollback

```bash
kubectl get pods
```

**Output:**

```text
nginx-deployment-5d44db985c-f9dnd   1/1     Running   0          8m42s
nginx-deployment-5d44db985c-xfr24   1/1     Running   0          8m47s
nginx-deployment-5d44db985c-zzj27   1/1     Running   0          8m40s
```

**Explanation:**

* All Pods are healthy and running
* ReplicaSet hash `5d44db985c` corresponds to the latest revision
* Confirms the buggy version is currently active

---

### 3. Confirm Current Kubernetes Context

```bash
kubectl config current-context
```

**Output:**

```text
kind-notecloud
```

**Explanation:**

* Confirms `kubectl` is pointing to the intended cluster

---

### 4. List Available Contexts

```bash
kubectl config get-contexts
```

**Output:**

```text
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         kind-notecloud   kind-notecloud   kind-notecloud
```

**Explanation:**

* `*` indicates the active context
* Confirms there is no ambiguity about the cluster being used

---

### 5. Verify Cluster Connectivity

```bash
kubectl cluster-info
```

**Output:**

```text
Kubernetes control plane is running at https://notecloud-control-plane:6443
CoreDNS is running at https://notecloud-control-plane:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

**Explanation:**

* Confirms the Kubernetes API server is reachable

---

### 6. Verify Node Details

```bash
kubectl get nodes -o wide
```

**Output:**

```text
notecloud-control-plane   Ready    control-plane   29m   v1.27.16-1+f5da3b717fc217   172.17.0.2   <none>   Ubuntu 23.10   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

**Explanation:**

* Node is in `Ready` state
* Confirms correct node and healthy cluster state

---

### 7. Confirm Deployment Presence Across Namespaces

```bash
kubectl get deployments -A
```

**Output:**

```text
default              nginx-deployment         3/3     3            3           12m
kube-system          coredns                  2/2     2            2           30m
local-path-storage   local-path-provisioner   1/1     1            1           30m
```

**Explanation:**

* Confirms the Deployment exists in the `default` namespace
* Confirms healthy state before rollback

---

## Rollback Action

### 8. Perform Rollback to Previous Revision

```bash
kubectl rollout undo deployment/nginx-deployment
```

**Output:**

```text
deployment.apps/nginx-deployment rolled back
```

**Explanation:**

* Kubernetes reverted from revision 2 to revision 1
* Previous ReplicaSet was reactivated
* New Pods were created using the previous image

---

## Post-Checks (Executed Exactly as Run)

### 9. Verify Pods After Rollback

```bash
kubectl get pods
```

**Output:**

```text
nginx-deployment-989f57c54-cc89r   1/1     Running   0          17s
nginx-deployment-989f57c54-d8q2m   1/1     Running   0          20s
nginx-deployment-989f57c54-f47vh   1/1     Running   0          21s
```

**Explanation:**

* New ReplicaSet hash `989f57c54` confirms rollback
* All Pods are running and healthy

---

### 10. Reconfirm Cluster and Node State

```bash
kubectl cluster-info
kubectl get nodes -o wide
```

**Output:**

```text
Kubernetes control plane is running at https://notecloud-control-plane:6443
notecloud-control-plane   Ready    control-plane   32m   v1.27.16-1+f5da3b717fc217   172.17.0.2   <none>   Ubuntu 23.10   5.4.0-1106-gcp   containerd://1.7.1-2-g8f682ed69
```

**Explanation:**

* Confirms cluster stability after rollback

---

### 11. Confirm Rollout Completion

```bash
kubectl rollout status deployment/nginx-deployment
```

**Output:**

```text
deployment "nginx-deployment" successfully rolled out
```

**Explanation:**

* Rollback completed successfully
* Deployment is fully reconciled

---

## FINAL STATE CONFIRMATION

| Check                                  | Status |
| -------------------------------------- | ------ |
| Correct cluster (kind-notecloud)       | ✅      |
| Correct node (notecloud-control-plane) | ✅      |
| Rollback executed                      | ✅      |
| Previous revision restored             | ✅      |
| All Pods running                       | ✅      |
| Deployment stable                      | ✅      |

---

## Interview-Grade Summary (Exact)

> “I verified the active Kubernetes context, node readiness, deployment history, and namespace before performing the rollback. I then used `kubectl rollout undo` to revert the Deployment to the previous revision, monitored the rollout status, and confirmed that new Pods were created and running successfully.”

---

**Status:** Task completed successfully with full validation

