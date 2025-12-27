# TaskAndResolution.md

## Task Overview

The objective of this task was to configure **persistent storage for a web application** on a Kubernetes cluster using **PersistentVolume (PV)** and **PersistentVolumeClaim (PVC)**, deploy an **nginx Pod** that serves content from the persistent storage, and expose it externally using a **NodePort Service**.

All actions below were performed from the **host**.

---

## Requirements

* Create a PersistentVolume named `pv-devops`

* StorageClass: `manual`

* Capacity: `3Gi`

* Access Mode: `ReadWriteOnce`

* Volume type: `hostPath`

* Path: `/mnt/dba`

* Create a PersistentVolumeClaim named `pvc-devops`

* StorageClass: `manual`

* Requested storage: `2Gi`

* Access Mode: `ReadWriteOnce`

* Create a Pod named `pod-devops`

* Container name: `container-devops`

* Image: `nginx:latest`

* Mount the PVC at nginx document root `/usr/share/nginx/html`

* Create a NodePort Service named `web-devops`

* NodePort: `30008`

---

## Step 1: Create PersistentVolume

```bash
kubectl apply -f pv-devops.yaml
```

### pv-devops.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-devops
spec:
  storageClassName: manual
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/dba
```

### Verification

```bash
kubectl get pv
```

```text
pv-devops   3Gi   RWO   Retain   Available   manual
```

---

## Step 2: Create PersistentVolumeClaim

```bash
kubectl apply -f pvc-devops.yaml
```

### pvc-devops.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-devops
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

### Verification

```bash
kubectl get pv,pvc
```

```text
pv-devops    Bound   pvc-devops   3Gi   RWO   manual
pvc-devops   Bound   pv-devops    3Gi   RWO   manual
```

---

## Step 3: Create Pod Using PVC

```bash
kubectl apply -f pod-devops.yaml
```

### pod-devops.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-devops
spec:
  containers:
  - name: container-devops
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: web-storage
  volumes:
  - name: web-storage
    persistentVolumeClaim:
      claimName: pvc-devops
```

### Verification

```bash
kubectl get pods
```

```text
pod-devops   1/1   Running
```

---

## Step 4: Create NodePort Service

```bash
kubectl label pod pod-devops app=web-devops
kubectl apply -f web-devops-service.yaml
```

### web-devops-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-devops
spec:
  type: NodePort
  selector:
    app: web-devops
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
```

### Verification

```bash
kubectl get svc web-devops
```

```text
web-devops   NodePort   80:30008/TCP
```

---

## Step 5: 403 Forbidden Issue & Resolution

### Issue Observed

Accessing the application returned:

```text
403 Forbidden
nginx/1.29.4
```

### Root Cause

* nginx document root was backed by an **empty PersistentVolume**
* No `index.html` file existed

### Fix Applied

```bash
kubectl exec -it pod-devops -- /bin/sh
echo "<h1>Welcome to DevOps Persistent Volume</h1>" > /usr/share/nginx/html/index.html
exit
```

### Final Verification

```bash
curl http://<NODE-IP>:30008
```

```html
<h1>Welcome to DevOps Persistent Volume</h1>
```

---

## Scenario-Based Interview Questions & Answers

### L1 – Junior DevOps Engineer

**Q1. Why do we need a PersistentVolume?**
To store data that must persist beyond the lifecycle of a Pod.

**Q2. What does ReadWriteOnce mean?**
The volume can be mounted as read-write by a single node at a time.

---

### L2 – Mid-Level DevOps Engineer

**Q3. Why was 403 Forbidden returned by nginx?**
Because the document root was empty and nginx could not find an index file.

**Q4. How does PVC bind to PV?**
Based on matching storage class, access mode, and requested capacity.

---

### L3 – Senior DevOps Engineer

**Q5. Why is hostPath not recommended in production?**
It ties storage to a single node and lacks durability and portability.

**Q6. How would you make this setup production-ready?**
Use dynamic provisioning with cloud-backed storage and Deployments instead of Pods.

---

### L4 – Architect Level

**Q7. How would this architecture change in cloud environments?**
Use managed storage services, Ingress controllers, and high-availability designs.

---

## AWS EKS Mapping & Real-World Incident RCAs

### Kubernetes to AWS Mapping

| Kubernetes  | AWS Equivalent    |
| ----------- | ----------------- |
| hostPath PV | Amazon EBS Volume |
| PVC         | EBS Volume Claim  |
| Pod         | Pod on EKS Node   |
| NodePort    | ALB / NLB         |

---

### AWS Scenario 1: Application Data Lost After Pod Restart

**Root Cause:**
Used ephemeral storage instead of EBS-backed PVC.

**Resolution:**
Migrated to EBS CSI driver with persistent volumes.

---

### AWS Scenario 2: ALB Returns 403 Error

**Root Cause:**
ALB routed traffic to Pods serving empty document root.

**Resolution:**
Ensured application artifacts were stored on persistent volumes.

---

### AWS Scenario 3: Pod Scheduling Failure

**Root Cause:**
EBS volume attached to wrong AZ.

**Resolution:**
Ensured node groups and volumes were in the same Availability Zone.

---

## Final Conclusion

This task demonstrated end-to-end persistent storage integration in Kubernetes, including troubleshooting application-level errors and mapping the solution to real-world AWS EKS architectures. It highlights the importance of proper storage design, validation, and cloud-native best practices.

