# Task and Resolution: Kubernetes Pod Resource Limits

---

## Task Description

The DevOps team identified performance issues in Kubernetes-hosted applications due to uncontrolled resource usage. The objective of this task is to **create a Kubernetes Pod with defined CPU and memory requests and limits** to ensure predictable performance and cluster stability.

### Requirements

* **Pod Name**: `httpd-pod`
* **Container Name**: `httpd-container`
* **Image**: `httpd:latest`
* **Resource Requests**:

  * Memory: `15Mi`
  * CPU: `100m`
* **Resource Limits**:

  * Memory: `20Mi`
  * CPU: `100m`

> Note: `kubectl` is preconfigured on the `jump_host` to access the Kubernetes cluster.

---

## Resolution Steps

### Step 1: Create the Pod Manifest

Create a YAML file named `httpd-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      resources:
        requests:
          memory: "15Mi"
          cpu: "100m"
        limits:
          memory: "20Mi"
          cpu: "100m"
```

**Explanation**:

* `apiVersion: v1` and `kind: Pod` define a basic Pod object
* `resources.requests` specify the **minimum guaranteed resources** for scheduling
* `resources.limits` enforce the **maximum allowed usage** at runtime

---

### Step 2: Apply the Pod Configuration

Run the following command:

```bash
kubectl apply -f httpd-pod.yaml
```

**Output**:

```text
pod/httpd-pod created
```

**Explanation**:

* This command submits the manifest to the Kubernetes API server
* The scheduler places the Pod on a node that can satisfy the resource requests

---

### Step 3: Verify Pod Status

```bash
kubectl get pod httpd-pod
```

**Output**:

```text
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          1m
```

**Explanation**:

* `READY 1/1` confirms the container is healthy
* `STATUS Running` indicates the Pod is active

---

### Step 4: Describe the Pod (Detailed Validation)

```bash
kubectl describe pod httpd-pod
```

**Full Output**:

```text
Name:             httpd-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       <timestamp>
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               <pod-ip>
IPs:
  IP:  <pod-ip>
Containers:
  httpd-container:
    Container ID:   containerd://<container-id>
    Image:          httpd:latest
    Image ID:       docker.io/library/httpd@sha256:<image-digest>
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      <timestamp>
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  20Mi
    Requests:
      cpu:        100m
      memory:     15Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pd5xv (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-pd5xv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  59s   default-scheduler  Successfully assigned default/httpd-pod to kodekloud-control-plane
  Normal  Pulling    58s   kubelet            Pulling image "httpd:latest"
  Normal  Pulled     53s   kubelet            Successfully pulled image "httpd:latest" in 4.857105495s (4.857122944s including waiting)
  Normal  Created    53s   kubelet            Created container httpd-container
  Normal  Started    52s   kubelet            Started container httpd-container
```

**Explanation**:

* All Pod conditions are `True`, confirming healthy initialization
* `QoS Class: Burstable` is assigned because memory request and limit differ
* Events show a successful lifecycle: scheduled → image pulled → container started

---

## Technical Analysis

### Why QoS Class is Burstable

* CPU: request = limit (`100m`)
* Memory: request (`15Mi`) < limit (`20Mi`)

Since **all resources do not have identical requests and limits**, Kubernetes assigns the **Burstable** QoS class.

### Runtime Behavior

* CPU usage above `100m` will be **throttled**
* Memory usage above `20Mi` will cause the container to be **OOMKilled**

---

## Final Outcome

* Pod created successfully
* Resource requests and limits enforced
* Pod running in healthy state
* Configuration meets production standards

---

## Key Takeaways (Interview Ready)

* `requests` are used for scheduling
* `limits` are enforced at runtime
* Memory limits trigger OOM kills
* QoS class impacts eviction priority

---

**Status**: Task completed and validated successfully

