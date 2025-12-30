# TaskAndResolution.md

## Task Title

Deploying and Verifying Redis as an In-Memory Cache on Kubernetes

---

## Task Objective

Deploy Redis on a Kubernetes cluster for testing purposes using a ConfigMap-based configuration, validate in-memory settings, volume mounts, CPU requests, and ensure Redis is fully operational.

---

## Environment Details

* Kubernetes Cluster: Preconfigured single-node control-plane lab
* Namespace: default
* Access Node: host
* Operating System (node): Ubuntu 23.10
* Kubernetes Version: v1.27.x
* Container Runtime: containerd
* Tool Used: kubectl

---

## Kubernetes Resources Created (Names & Configuration)

### 1. ConfigMap

* **Name:** my-redis-config
* **Type:** ConfigMap
* **Key:** redis-config
* **Value:** maxmemory 2mb
* **Purpose:** Externalized Redis memory configuration

### 2. Deployment

* **Name:** redis-deployment
* **Replicas:** 1
* **Strategy:** RollingUpdate
* **Label Selector:** app=redis

**Container Details**

* **Container Name:** redis-container
* **Image:** redis:alpine
* **Command:** redis-server /redis-master/redis-config
* **Exposed Port:** 6379/TCP

**Resource Configuration**

* **CPU Request:** 1 vCPU
* **Memory Request:** Not specified (Burstable QoS)

**Volumes**

* **data:** emptyDir mounted at /redis-master-data
* **redis-config:** ConfigMap volume mounted at /redis-master

### 3. Pod (Auto-created by Deployment)

* **Generated Name:** redis-deployment-<hash>
* **QoS Class:** Burstable
* **Restart Policy:** Always

---

## Step 1: Create ConfigMap for Redis Configuration

### Command

```bash
kubectl create configmap my-redis-config --from-literal=redis-config="maxmemory 2mb"
```

### Output

```text
configmap/my-redis-config created
```

### Explanation

Creates a ConfigMap named `my-redis-config` with Redis configuration setting `maxmemory 2mb`, which limits Redis memory usage.

---

## Step 2: Verify ConfigMap Content

### Command

```bash
kubectl describe configmaps my-redis-config
```

### Output

```text
Name:         my-redis-config
Namespace:    default
Data
====
redis-config:
----
maxmemory 2mb
```

### Explanation

Confirms that the ConfigMap contains the expected Redis configuration.

---

## Step 3: Create Redis Deployment Manifest

### File Creation

```bash
vi redis-deployment.yaml
```

### Manifest Content

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        command:
          - redis-server
          - /redis-master/redis-config
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
```

### Explanation

Defines a single-replica Redis deployment using `redis:alpine`, mounts configuration and data volumes, requests 1 CPU, and exposes port 6379.

---

## Step 4: Deploy Redis

### Command

```bash
kubectl apply -f redis-deployment.yaml
```

### Output

```text
deployment.apps/redis-deployment created
```

### Explanation

Creates the Redis Deployment in the cluster.

---

## Step 5: Monitor Deployment and Pod Status

### Command

```bash
kubectl get deployment redis-deployment
```

### Output (Initial)

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1            0           2m
```

### Explanation

Indicates the Deployment exists but Pod is not yet available.

---

## Step 6: Check Pod Status

### Command

```bash
kubectl get pods -l app=redis
```

### Output (Transient)

```text
redis-deployment-6849b9469-f4l49   0/1   ImagePullBackOff
```

### Explanation

ImagePullBackOff occurred due to transient Docker Hub image pull failure.

---

## Step 7: Describe Pod for Root Cause Analysis

### Command

```bash
kubectl describe pod redis-deployment-6849b9469-f4l49
```

### Output (Key Sections)

```text
Image: redis:alpine
State: Running
Warning: ImagePullBackOff (earlier)
```

### Explanation

Shows initial pull failures followed by successful image pull and container startup.

---

## Step 8: Confirm Deployment Is Now Healthy

### Command

```bash
kubectl get deployment redis-deployment
```

### Output

```text
redis-deployment   1/1   1   1
```

### Explanation

Confirms Redis deployment is fully available.

---

## Step 9: Verify Redis Configuration (maxmemory)

### Command

```bash
kubectl exec -it redis-deployment-6849b9469-f4l49 -- redis-cli CONFIG GET maxmemory
```

### Output

```text
1) "maxmemory"
2) "2097152"
```

### Explanation

Validates that Redis is enforcing the `maxmemory 2mb` configuration from the ConfigMap.

---

## Step 10: Verify ConfigMap Mount

### Command

```bash
kubectl exec -it redis-deployment-6849b9469-f4l49 -- ls /redis-master
```

### Output

```text
redis-config
```

### Explanation

Confirms ConfigMap file is mounted inside the container.

---

## Step 11: Verify Redis Port Listening

### Command

```bash
kubectl exec -it redis-deployment-6849b9469-f4l49 -- netstat -ntupl | grep 6379
```

### Output

```text
LISTEN 0.0.0.0:6379
```

### Explanation

Confirms Redis is listening on the default port 6379.

---

## Step 12: Verify Redis Connectivity

### Command

```bash
kubectl exec -it redis-deployment-6849b9469-f4l49 -- redis-cli ping
```

### Output

```text
PONG
```

### Explanation

Confirms Redis responds to client requests.

---

## Step 13: Verify emptyDir Volume Writable

### Command

```bash
kubectl exec -it redis-deployment-6849b9469-f4l49 -- touch /redis-master-data/testfile
```

### Explanation

Confirms Redis data directory is writable.

---

## Step 14: Verify CPU Request and QoS Class

### Commands

```bash
kubectl describe pod redis-deployment-6849b9469-f4l49 | grep -A3 Requests
kubectl describe pod redis-deployment-6849b9469-f4l49 | grep QoS
```

### Output

```text
Requests:
  cpu: 1
QoS Class: Burstable
```

### Explanation

Confirms CPU request is honored and Pod has Burstable QoS.

---

# Scenario-Based Interview Questions & Answers

## L1 – Junior Level

**Q: What is a ConfigMap used for?**
A: To store non-sensitive configuration data separately from container images.

---

## L2 – Mid Level

**Q: Why use emptyDir for Redis?**
A: Redis is a cache; emptyDir provides fast ephemeral storage.

---

## L3 – Senior Level

**Q: Why did ImagePullBackOff resolve automatically?**
A: Kubernetes retries pulling images; transient network issues can self-heal.

---

## L4 – Architect Level

**Q: How would you run Redis in production on EKS?**
A: Use StatefulSets, PersistentVolumes (EBS), replicas, and managed Redis if possible.

---

# AWS Scenario Mapping & Real Incident RCAs

## Incident 1: Redis Pods Stuck in ImagePullBackOff (EKS)

* **Cause:** Docker Hub rate limits
* **Fix:** Mirror images to Amazon ECR

## Incident 2: Cache Evictions Due to Wrong maxmemory

* **Cause:** Missing Redis configuration
* **Fix:** ConfigMap-based config with validation

## Incident 3: Data Loss After Pod Restart

* **Cause:** emptyDir used in production
* **Fix:** Migrated Redis to StatefulSet with EBS-backed PVs

---

## Final Takeaways

* ConfigMaps externalize configuration
* emptyDir is suitable for cache, not persistence
* Always verify runtime config, not just YAML
* Kubernetes can recover from transient pull errors

---

**Document Status:** Complete, audit-ready, interview-ready

