# TaskAndResolution.md

## Task Title

Deploying Iron Gallery Application with Database Connectivity on Kubernetes

---

## Task Objective

Deploy a frontend application (Iron Gallery) and a backend database (MariaDB) on Kubernetes, expose services correctly, and establish connectivity between the frontend and database using Kubernetes Service DNS and environment variables.

---

## Environment Details

* Kubernetes Cluster: Preconfigured
* Namespace: `iron-namespace-datacenter`
* Access Node: host
* Tools: kubectl

---

## Step 1: Create Namespace

### Command

```bash
kubectl create namespace iron-namespace-datacenter
```

### Output

```text
namespace/iron-namespace-datacenter created
```

### Explanation

* Creates an isolated Kubernetes namespace for all Iron Gallery resources.
* Ensures logical separation from other workloads.

---

## Step 2: Deploy Iron Gallery (Frontend)

### Manifest: iron-gallery-deployment-datacenter.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-datacenter
  namespace: iron-namespace-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - name: iron-gallery-container-datacenter
        image: cloud/irongallery:2.0
        resources:
          limits:
            cpu: "50m"
            memory: "100Mi"
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html/data
        - name: images
          mountPath: /usr/share/nginx/html/uploads
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}
```

### Apply Deployment

```bash
kubectl apply -f iron-gallery-deployment-datacenter.yml
```

### Output

```text
deployment.apps/iron-gallery-deployment-datacenter created
```

### Explanation

* Deploys the Iron Gallery frontend.
* Uses emptyDir volumes for configuration and uploads.
* Resource limits protect cluster stability.

---

## Step 3: Deploy Iron DB (MariaDB)

### Manifest: iron-db-deployment-datacenter.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-datacenter
  namespace: iron-namespace-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db-container-datacenter
        image: cloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: database_host
        - name: MYSQL_ROOT_PASSWORD
          value: R00t@123#Db
        - name: MYSQL_PASSWORD
          value: Us3r@456#Db
        - name: MYSQL_USER
          value: ironuser
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}
```

### Apply Deployment

```bash
kubectl apply -f iron-db-deployment-datacenter.yml
```

### Output

```text
deployment.apps/iron-db-deployment-datacenter created
```

### Explanation

* Deploys MariaDB with required environment variables.
* Uses emptyDir for ephemeral storage (lab use).

---

## Step 4: Create Services

### Iron DB Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  selector:
    db: mariadb
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
  type: ClusterIP
```

```bash
kubectl apply -f iron-db-service-datacenter.yml
```

### Output

```text
service/iron-db-service-datacenter created
```

---

### Iron Gallery Service (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  selector:
    run: iron-gallery
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 32678
  type: NodePort
```

```bash
kubectl apply -f iron-gallery-service-datacenter.yml
```

### Output

```text
service/iron-gallery-service-datacenter created
```

---

## Step 5: Verify Resources

```bash
kubectl get all -n iron-namespace-datacenter
```

### Output (excerpt)

```text
pod/iron-db-deployment-datacenter-xxxxx        Running
pod/iron-gallery-deployment-datacenter-yyyyy  Running
service/iron-db-service-datacenter             ClusterIP
service/iron-gallery-service-datacenter        NodePort
```

### Explanation

* Confirms Pods, Deployments, and Services are active.

---

## Step 6: Connect Frontend to Database

### Edit Gallery Deployment

```bash
kubectl edit deployment iron-gallery-deployment-datacenter -n iron-namespace-datacenter
```

### Added Environment Variables

```yaml
env:
- name: DB_HOST
  value: iron-db-service-datacenter
- name: DB_DATABASE
  value: database_host
- name: DB_USERNAME
  value: ironuser
- name: DB_PASSWORD
  value: Us3r@456#Db
```

### Output

```text
deployment.apps/iron-gallery-deployment-datacenter edited
```

### Explanation

* Injects DB connection details into frontend container.
* Uses Service DNS for connectivity.

---

## Step 7: Verify Environment Variables

```bash
kubectl get pods -n iron-namespace-datacenter
kubectl exec -it <gallery-pod> -n iron-namespace-datacenter -- env | grep DB_
```

### Output

```text
DB_HOST=iron-db-service-datacenter
DB_DATABASE=database_host
DB_USERNAME=ironuser
DB_PASSWORD=Us3r@456#Db
```

---

## Step 8: Access Application

```text
http://<NodeIP>:32678
```

* Installation page loads and connects to DB successfully.

---

# Scenario-Based Interview Questions & Answers

---

## L1 – Junior Level

**Q:** Why do we use Services instead of Pod IPs?
**A:** Pod IPs are ephemeral; Services provide stable endpoints.

---

## L2 – Mid Level

**Q:** Why is ClusterIP used for DB service?
**A:** DB should be accessible only inside the cluster.

---

## L3 – Senior Level

**Q:** What happens if the DB Pod restarts?
**A:** Service DNS remains the same, so connectivity is preserved.

---

## L4 – Architect Level

**Q:** How would you secure DB credentials in production?
**A:** Use Kubernetes Secrets or external secret managers.

---

# AWS EKS – Scenario Mapping & Incident RCAs

---

## Incident 1: Frontend Unable to Reach DB

* **Cause:** Hardcoded Pod IP
* **Fix:** Switched to Service DNS

## Incident 2: Credential Leakage

* **Cause:** Plain-text env vars in manifests
* **Fix:** Migrated to AWS Secrets Manager + IRSA

## Incident 3: Data Loss

* **Cause:** emptyDir used in production
* **Fix:** Migrated to EBS-backed PersistentVolumes

---

## Final Takeaways

* Kubernetes Services enable stable networking
* Environment variables are common for configuration
* Separation of frontend and DB is critical
* Production requires secure secret and storage handling

---

**Document Status:** Complete, interview-ready, and production-aligned

