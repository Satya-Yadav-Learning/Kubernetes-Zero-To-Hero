# TaskAndResolution.md

## Task Overview

Provision a MySQL database on Kubernetes using PersistentVolumes, PersistentVolumeClaims, Secrets, a Deployment, and a NodePort Service. Diagnose and resolve a `CreateContainerConfigError` caused by missing Secrets. Provide full configuration details, command executions, outputs, and a comprehensive RCA.

---

## Kubernetes Resources Names & Configuration

| Resource Type         | Name             | Key Configuration                                                     |
| --------------------- | ---------------- | --------------------------------------------------------------------- |
| PersistentVolume      | mysql-pv         | 250Mi, RWO, Retain, storageClass: mysql-sc, hostPath: /mnt/mysql-data |
| PersistentVolumeClaim | mysql-pv-claim   | 250Mi request, RWO, storageClass: mysql-sc                            |
| Secret                | mysql-root-pass  | key: password                                                         |
| Secret                | mysql-user-pass  | keys: username, password                                              |
| Secret                | mysql-db-url     | key: database                                                         |
| Deployment            | mysql-deployment | image: mysql:8.0, mount: /var/lib/mysql, env from secrets             |
| Service               | mysql            | NodePort 30007 → 3306                                                 |

---

## Environment

* Kubernetes Cluster: cloud-managed Kubernetes cluster
* Namespace: default
* Container Image: mysql:8.0
* Storage Type: hostPath (cloud  cluster)
* Access Mode: ReadWriteOnce
* Service Type: NodePort (30007)

---

## Step-by-Step Implementation

### 1. PersistentVolume (mysql-pv)

**Configuration**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: mysql-sc
  hostPath:
    path: /mnt/mysql-data
```

**Command**

```bash
kubectl apply -f mysql-pv.yml
```

**Output**

```
persistentvolume/mysql-pv created
```

---

### 2. PersistentVolumeClaim (mysql-pv-claim)

**Configuration**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
  storageClassName: mysql-sc
```

**Command**

```bash
kubectl apply -f mysql-pv-claim.yml
```

**Output**

```
persistentvolumeclaim/mysql-pv-claim created
```

**Verification**

```bash
kubectl get pv
kubectl get pvc
```

**Output**

```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   mysql-sc       94s

NAME             STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            mysql-sc       68s
```

---

### 3. Secrets

#### mysql-root-pass

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: YUIidhb667
```

#### mysql-user-pass

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: cloud_top
  password: YchZHRcLkL
```

#### mysql-db-url

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: cloud_db10
```

**Commands**

```bash
kubectl apply -f mysql-root-pass.yml
kubectl apply -f mysql-user-pass.yml
kubectl apply -f mysql-db-url.yml
```

**Output**

```
secret/mysql-root-pass created
secret/mysql-user-pass created
secret/mysql-db-url created
```

---

### 4. Deployment (mysql-deployment)

**Configuration**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      els:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

**Command**

```bash
kubectl apply -f mysql-deployment.yml
```

---

### 5. Service (NodePort)

**Configuration**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
    nodePort: 30007
```

**Command**

```bash
kubectl apply -f mysql-service.yaml
```

---

## Failure & Troubleshooting

### Symptom

```bash
kubectl get pods -l app=mysql
```

```
CreateContainerConfigError
```

### Root Cause

Missing secret `mysql-root-pass` referenced by Deployment.

### Evidence

```
Error: secret "mysql-root-pass" not found
```

### Resolution

```bash
kubectl create secret generic mysql-root-pass \
  --from-literal=password=YUIidhb667
kubectl delete pod -l app=mysql
```

### Final Verification

```bash
kubectl get pods -l app=mysql
kubectl logs -l app=mysql --tail=10
kubectl get endpoints mysql
```

---

## Final State

* Deployment: 1/1 Ready
* Pod: Running
* MySQL: Ready for connections
* Service Endpoint: Active

---

## Scenario-Based Interview Q&A

### L1 – Junior

**Q:** What caused `CreateContainerConfigError`?
**A:** A required Secret referenced by the Pod did not exist.

### L2 – Engineer

**Q:** Why were logs unavaile?
**A:** Container never started; failure occurred during config validation.

### L3 – Senior

**Q:** How does Kubernetes validate secrets?
**A:** At container creation time; missing non-optional secrets block startup.

### L4 – Architect

**Q:** How would you prevent this in production?
**A:** GitOps ordering, admission controllers, secret validation pipelines.

---

## AWS Scenario Mapping

### EKS Mapping

* PV → EBS Volume
* PVC → EBS CSI Claim
* Secret → AWS Secrets Manager
* NodePort → NLB/ALB

### AWS Incident RCA

**Incident:** RDS migration app crash on EKS
**Root Cause:** Secret rotation removed DB credentials
**Impact:** Pods stuck in `CreateContainerConfigError`
**Fix:** Restore secret, add pre-deploy validation, integrate External Secrets Operator

---

## Conclusion

This task demonstrates full lifecycle Kubernetes operations, failure diagnosis, and production-grade recovery with direct AWS relevance.

