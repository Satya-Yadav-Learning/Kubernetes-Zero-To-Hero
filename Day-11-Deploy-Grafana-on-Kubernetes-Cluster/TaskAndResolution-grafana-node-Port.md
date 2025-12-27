# TaskAndResolution.md

## Task

Deploy **Grafana** on a Kubernetes cluster and expose it externally using a **NodePort Service**, ensuring that the Grafana **login page is accessible**, without making any configuration changes inside the Grafana application.

All steps below were executed from the **host**.

---

## Requirements

* Create a Deployment named `grafana-deployment-devops`
* Use any Grafana image
* Create a NodePort Service
* NodePort must be **32000**
* Validate access to Grafana login page

---

## Step 1: Create Grafana Deployment Manifest

```bash
host ~$ vi grafana-deployment.yaml
```

### grafana-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-devops
  labels:
    app: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana-container
          image: grafana/grafana:latest
          ports:
            - containerPort: 3000
```

### Explanation

* Grafana runs on port **3000** by default
* One replica is sufficient for access validation
* No persistence or configuration changes required

---

## Step 2: Create Grafana NodePort Service

```bash
host ~$ vi grafana-service.yaml
```

### grafana-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-devops
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32000
```

### Explanation

* NodePort exposes Grafana externally
* Port **32000** is opened on all cluster nodes

---

## Step 3: Apply Deployment

```bash
host ~$ kubectl apply -f grafana-deployment.yaml
deployment.apps/grafana-deployment-devops created
```

### Explanation

* Deployment controller creates a ReplicaSet
* ReplicaSet launches Grafana Pod

---

## Step 4: Apply Service

```bash
host ~$ kubectl apply -f grafana-service.yaml
service/grafana-service-devops created
```

---

## Step 5: Verify Pod Status

```bash
host ~$ kubectl get pods
```

### Output (Initial)

```text
NAME                                         READY   STATUS              RESTARTS   AGE
grafana-deployment-devops-664fcfc669-2zxz8   0/1     ContainerCreating   0          26s
```

### Output (After Image Pull)

```text
NAME                                         READY   STATUS    RESTARTS   AGE
grafana-deployment-devops-664fcfc669-2zxz8   1/1     Running   0          35s
```

### Explanation

* Image pull completed
* Container started successfully
* Grafana process is running

---

## Step 6: Verify Service

```bash
host ~$ kubectl get svc grafana-service-devops
```

### Output

```text
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana-service-devops   NodePort   10.96.144.124   <none>        3000:32000/TCP   38s
```

### Explanation

* ClusterIP is internal
* NodePort **32000** is active for external access

---

## Step 7: Verify Node Information

```bash
host ~$ kubectl get nodes -o wide
```

### Output

```text
NAME                      STATUS   ROLES           AGE   VERSION        INTERNAL-IP   EXTERNAL-IP
notecloud-control-plane   Ready    control-plane   17m   v1.27.16       172.17.0.2    <none>
```

### Explanation

* Node is in Ready state
* Node IP is used with NodePort for access

---

## Step 8: Access Grafana Login Page

### Lab Environment URL

```text
https://32000-port-xxxx.labs.notecloud.com/login
```

### Observed Page

```text
Welcome to Grafana
Email or username
Password
Log in
```

### Confirmation

* NodePort routing works correctly
* Grafana UI is reachable
* No internal configuration was modified

---

## Scenario-Based Interview Questions & Answers

### L1 – Junior DevOps Engineer

**Q1. Why does Grafana show `Running` status?**
Because the container started successfully and the Grafana process is active.

**Q2. What is the purpose of NodePort?**
It exposes the application on a fixed port on each node.

---

### L2 – Mid-Level DevOps Engineer

**Q3. How does traffic reach the Grafana Pod?**
Client → NodeIP:32000 → NodePort Service → ClusterIP → Pod:3000.

**Q4. Why is ClusterIP not used directly for browser access?**
Because ClusterIP is internal-only and not routable externally.

---

### L3 – Senior DevOps Engineer

**Q5. What are the limitations of NodePort in production?**
Fixed port range, no TLS termination, manual DNS handling.

**Q6. What is the recommended production alternative?**
Use LoadBalancer Service or Ingress with TLS.

---

### L4 – Architect Level

**Q7. How would you design this in a cloud-native way?**
Use ALB/Ingress, autoscaling, and persistent storage for Grafana.

---

## AWS EKS Mapping

### Kubernetes to AWS Mapping

| Kubernetes Component | AWS Equivalent               |
| -------------------- | ---------------------------- |
| Pod                  | Container on EC2 worker node |
| Deployment           | Managed ReplicaSet on EKS    |
| NodePort Service     | ALB/NLB                      |

---

## AWS Scenario-Based Interview Q&A

**Q8. What happens when a LoadBalancer Service is created in EKS?**
AWS provisions an ELB/ALB and maps it to the Kubernetes Service.

**Q9. How is Grafana made highly available in AWS?**
By using multiple replicas across AZs and an ALB.

---

## Real AWS Incident – RCA Examples

### Incident 1: Grafana UI Not Accessible

**Root Cause:**
Security Group did not allow inbound traffic on NodePort/ALB port.

**Resolution:**
Updated security group rules to allow required ports.

---

### Incident 2: Pod Restart Loop

**Root Cause:**
Insufficient memory caused OOMKilled events.

**Resolution:**
Increased resource limits and enabled monitoring.

---

### Incident 3: LoadBalancer Not Provisioned

**Root Cause:**
IAM role attached to worker nodes lacked ELB permissions.

**Resolution:**
Attached required AWS managed IAM policies.

---

## Final Summary

Grafana was successfully deployed on Kubernetes and exposed via a NodePort Service. The setup validates Kubernetes service networking concepts and maps cleanly to AWS EKS production architectures using managed LoadBalancers, autoscaling, and fault-tolerant design principles.

