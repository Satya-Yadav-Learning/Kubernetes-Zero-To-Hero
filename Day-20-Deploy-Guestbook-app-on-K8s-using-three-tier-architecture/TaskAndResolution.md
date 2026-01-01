# Guestbook Application – TaskAndResolution.md

## 1. Task Overview

Deploy a **Guestbook application** on Kubernetes using a **three-tier architecture**:

* **Redis Master** (write backend)
* **Redis Slaves** (read replicas)
* **Frontend (PHP-based UI)** exposed via NodePort

The task includes:

* Creating Deployments and Services
* Resource requests configuration
* Environment variable–based service discovery
* Troubleshooting a real `ImagePullBackOff` issue
* Applying a production-grade fix

---

## 2. Kubernetes Resources Summary

| Component    | Resource Type      | Name         |
| ------------ | ------------------ | ------------ |
| Redis Master | Deployment         | redis-master |
| Redis Master | Service            | redis-master |
| Redis Slave  | Deployment         | redis-slave  |
| Redis Slave  | Service            | redis-slave  |
| Frontend     | Deployment         | frontend     |
| Frontend     | Service (NodePort) | frontend     |

---

## 3. Backend Tier Configuration

### 3.1 Redis Master Deployment

**File:** `Redis-Master-Deployment.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
        - name: master-redis-natural
          image: redis
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
```

**Command**

```bash
kubectl apply -f Redis-Master-Deployment.yml
```

**Output**

```
deployment.apps/redis-master created
```

---

### 3.2 Redis Master Service

**File:** `Redis-Master-Service.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
spec:
  selector:
    app: redis
    role: master
  ports:
    - port: 6379
      targetPort: 6379
```

**Command**

```bash
kubectl apply -f Redis-Master-Service.yml
```

**Output**

```
service/redis-master created
```

---

### 3.3 Redis Slave Deployment

**File:** `Redis-Slave-Deployment.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
        - name: slave-redis-natural
          image: gcr.io/google_samples/gb-redisslave:v3
          ports:
            - containerPort: 6379
          env:
            - name: GET_HOSTS_FROM
              value: dns
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
```

**Command**

```bash
kubectl apply -f Redis-Slave-Deployment.yml
```

**Output**

```
deployment.apps/redis-slave created
```

---

### 3.4 Redis Slave Service

**File:** `Redis-Slave-Service.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
spec:
  selector:
    app: redis
    role: slave
  ports:
    - port: 6379
      targetPort: 6379
```

**Command**

```bash
kubectl apply -f Redis-Slave-Service.yml
```

**Output**

```
service/redis-slave created
```

---

## 4. Frontend Tier Configuration

### 4.1 Frontend Deployment

**File:** `Frontend-Deployment.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
        - name: php-redis-natural
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          ports:
            - containerPort: 80
          env:
            - name: GET_HOSTS_FROM
              value: dns
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
```

**Command**

```bash
kubectl apply -f Frontend-Deployment.yml
```

**Output**

```
deployment.apps/frontend created
```

---

### 4.2 Frontend NodePort Service

**File:** `Frontend-Service-NodePort.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: guestbook
    tier: frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30009
```

**Command**

```bash
kubectl apply -f Frontend-Service-NodePort.yml
```

**Output**

```
service/frontend created
```

---

## 5. Issue Encountered – ImagePullBackOff

### Symptom

```bash
kubectl get pods
```

```
redis-master-xxxx   0/1   ImagePullBackOff
```

### Investigation

```bash
kubectl describe pod redis-master-xxxx
```

**Key Event Output**

```
Failed to pull image "docker.io/library/redis:latest"
failed to read expected number of bytes: unexpected EOF
```

### Root Cause

* Redis master used Docker Hub image (`redis:latest`)
* Cluster had restricted/unstable Docker Hub access
* Image pull failed before container start

---

## 6. Resolution Applied

### Fix

Replaced Docker Hub image with Google-hosted Redis image.

```bash
kubectl set image deployment redis-master \
  master-redis-natural=k8s.gcr.io/redis:e2e
```

### Rollout Verification

```bash
kubectl rollout status deployment redis-master
```

```
deployment "redis-master" successfully rolled out
```

```bash
kubectl get pods
```

```
redis-master-xxxxx   1/1   Running
```

---

## 7. Final Validation

```bash
kubectl get deployments
kubectl get services
```

All components reached **Running / Available** state.

Guestbook UI accessible via:

```
http://<NodeIP>:30009
```

---

## 8. Scenario-Based Interview Questions & Answers

### L1 – Junior Level

**Q:** What caused `ImagePullBackOff`?
**A:** Kubernetes could not pull the container image from the registry.

### L2 – Engineer Level

**Q:** Why did `kubectl logs` not work?
**A:** The container never started, so no logs were generated.

### L3 – Senior Level

**Q:** Why did frontend load even when Redis master was down?
**A:** Frontend is stateless and does not require Redis for initial page rendering.

### L4 – Architect Level

**Q:** How would you prevent this issue in production?
**A:** Use private registries, pin image versions, validate image access during CI, and enforce admission policies.

---

## 9. AWS Mapping & Incident RCA

### AWS EKS Mapping

| Kubernetes         | AWS Equivalent                |
| ------------------ | ----------------------------- |
| Deployment         | ReplicaSet / ASG-managed Pods |
| Service (NodePort) | ALB / NLB                     |
| Redis              | ElastiCache                   |
| Image Registry     | Amazon ECR                    |

---

### Real AWS Incident RCA

**Incident:** Application outage after deployment

**Root Cause:** EKS nodes lost access to Docker Hub during peak traffic

**Impact:** Stateful services failed to start

**Resolution:** Migrated images to Amazon ECR and enforced image source validation

**Prevention:**

* Private ECR-only pulls
* CI validation
* Admission control policies

---

## 10. Conclusion

This task demonstrated real-world Kubernetes deployment, troubleshooting, and recovery practices aligned with production and AWS cloud environments.

