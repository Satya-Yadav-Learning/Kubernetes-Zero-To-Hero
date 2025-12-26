# TaskAndResolution.md

## Task

Deploy a highly available and scalable static website on a Kubernetes cluster using **nginx**, with:

* A Deployment having **3 replicas**
* An exposed **NodePort Service** on port **30011**

All steps below were executed from the **jumphost**.

---

## Step 1: Create Deployment Manifest

```bash
jumphost ~$ vi nginx-deployment.yaml
```

### nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

### Explanation

* `replicas: 3` ensures high availability
* `nginx:latest` explicitly defines the image tag
* Labels allow the Service to target the Pods

---

## Step 2: Create Service Manifest

```bash
jumphost ~$ vi nginx-service.yaml
```

### nginx-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30011
```

### Explanation

* `NodePort` exposes nginx externally
* Port `30011` is opened on all nodes
* Traffic is forwarded to Pod port `80`

---

## Step 3: Apply Deployment

```bash
jumphost ~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

### Explanation

* Deployment controller creates a ReplicaSet
* ReplicaSet launches 3 nginx Pods

---

## Step 4: Apply Service

```bash
jumphost ~$ kubectl apply -f nginx-service.yaml
service/nginx-service created
```

### Explanation

* Service is registered in Kubernetes
* NodePort 30011 becomes active

---

## Step 5: Verify Pods

```bash
jumphost ~$ kubectl get pods
```

### Output

```text
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5b58668cfc-lvnhh   1/1     Running   0          35s
nginx-deployment-5b58668cfc-rls52   1/1     Running   0          35s
nginx-deployment-5b58668cfc-sx89h   1/1     Running   0          35s
```

### Explanation

* All Pods are in `Running` state
* `1/1 READY` confirms nginx is healthy

---

## Step 6: Verify Deployment

```bash
jumphost ~$ kubectl get deployments
```

### Output

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           61s
```

### Explanation

* Desired replicas achieved
* Deployment is fully available

---

## Step 7: Verify Pod Details

```bash
jumphost ~$ kubectl get pods -o wide
```

### Output

```text
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE
nginx-deployment-5b58668cfc-lvnhh   1/1     Running   0          70s   10.244.0.7   notecloud-control-plane
nginx-deployment-5b58668cfc-rls52   1/1     Running   0          70s   10.244.0.5   notecloud-control-plane
nginx-deployment-5b58668cfc-sx89h   1/1     Running   0          70s   10.244.0.6   notecloud-control-plane
```

### Explanation

* Each Pod has a unique IP
* All Pods are scheduled successfully

---

## Step 8: Verify Service

```bash
jumphost ~$ kubectl get svc nginx-service
```

### Output

```text
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.27.34   <none>        80:30011/TCP   77s
```

### Explanation

* ClusterIP is internal-only
* NodePort 30011 exposes nginx externally

---

## Step 9: Access Validation

### nginx Default Page Output

```text
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working.
Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```

### Meaning

* nginx is running successfully
* Service routing is working correctly
* Traffic is reaching Pods end-to-end

---

## Correct Ways to Access nginx

### Option 1: Using Node IP + NodePort (Recommended)

```bash
curl http://<NODE-IP>:30011
```

**Example:**

```bash
curl http://notecloud-control-plane:30011
```

or

```bash
curl http://127.0.0.1:30011
```

*(if running directly on the node)*

---

### Option 2: Using ClusterIP (Internal Access Only)

```bash
curl http://10.96.27.34:80
```

---

## Final Result

* nginx Deployment with 3 replicas is running
* NodePort Service is exposing the application
* Static website is reachable externally

---

## Interview-Ready Summary

The Deployment ensures high availability through multiple replicas, while the NodePort Service exposes the nginx application externally by mapping port 30011 on each node to port 80 on the Pods.

---

## Scenario-Based Interview Questions & Answers (L1 → L4)

### L1 – Junior DevOps Engineer (Fundamentals)

**Q1. What was the goal of this task?**
**Answer:**
To deploy a static website using nginx on Kubernetes with high availability by running multiple replicas and exposing it externally using a NodePort Service.

**Q2. Why did we use a Deployment instead of a Pod?**
**Answer:**
A Deployment provides replica management, self-healing, and scaling. If a Pod crashes, Kubernetes automatically recreates it.

**Q3. What does `replicas: 3` achieve?**
**Answer:**
It ensures high availability. Traffic can still be served even if one Pod goes down.

**Q4. What is the purpose of a NodePort Service?**
**Answer:**
It exposes the application outside the cluster by opening a fixed port (30011) on every node.

---

### L2 – Mid-Level DevOps Engineer (Operations)

**Q5. How does traffic flow from browser to nginx Pod?**
**Answer:**
Browser → Node IP:30011 → NodePort Service → ClusterIP → kube-proxy → nginx Pod (port 80).

**Q6. How does Kubernetes load balance traffic across Pods?**
**Answer:**
kube-proxy distributes requests in a round-robin fashion across all healthy Pods selected by the Service.

**Q7. How would you scale this application?**
**Answer:**
By increasing replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

**Q8. What happens if one Pod crashes?**
**Answer:**
The Deployment controller detects the failure and recreates the Pod automatically.

---

### L3 – Senior DevOps Engineer (Design & Reliability)

**Q9. Why is `nginx:latest` not recommended in production?**
**Answer:**
The `latest` tag can change unexpectedly, leading to untested versions being deployed. Version pinning ensures reproducibility.

**Q10. What are the limitations of NodePort in production?**
**Answer:**

* Fixed port range (30000–32767)
* No TLS termination
* No advanced routing
* Manual DNS management

**Q11. How would you expose this application securely?**
**Answer:**
Use a LoadBalancer Service or Ingress with TLS termination.

**Q12. How would you ensure zero-downtime deployments?**
**Answer:**
By configuring rolling update strategies and readiness probes.

---

### L4 – Architect Level (System Design & Cloud Mapping)

**Q13. How would this design change in AWS EKS?**
**Answer:**
NodePort would be replaced by an AWS LoadBalancer Service or an Ingress backed by an ALB.

**Q14. How do you design for internet-scale traffic?**
**Answer:**

* Use ALB + Ingress Controller
* Auto Scaling Groups
* Horizontal Pod Autoscaler
* Multi-AZ deployment

**Q15. What production risks exist in this setup?**
**Answer:**

* No TLS
* No health probes
* Single cluster dependency

---

## AWS EKS / LoadBalancer Mapping

### Kubernetes → AWS Mapping

| Kubernetes Component | AWS Equivalent                       |
| -------------------- | ------------------------------------ |
| Deployment           | ReplicaSet + ASG-backed worker nodes |
| Pod                  | EC2 instance task (logical unit)     |
| Service (NodePort)   | Classic/NLB/ALB LoadBalancer         |
| kube-proxy           | AWS VPC networking                   |

### Recommended AWS EKS Design

* Use **Service type: LoadBalancer**
* AWS provisions an **Application Load Balancer (ALB)**
* ALB routes traffic to NodeGroups
* Nodes forward traffic to Pods

---

## AWS Scenario-Based Interview Q&A

**Q16. What happens when you create a LoadBalancer Service in EKS?**
**Answer:**
AWS automatically provisions an ELB/ALB and maps it to the Kubernetes Service.

**Q17. How is high availability achieved in AWS?**
**Answer:**
Through multi-AZ ALBs, Auto Scaling Groups, and multiple Pod replicas.

**Q18. How do you handle sudden traffic spikes?**
**Answer:**

* HPA scales Pods
* ASG scales nodes
* ALB distributes traffic

---

## Real AWS Incident – RCA Examples

### Incident 1: Application Down After Deployment

**Root Cause:**
Readiness probe missing; ALB sent traffic to unready Pods.

**Resolution:**
Added readiness and liveness probes.

---

### Incident 2: High Latency During Traffic Spike

**Root Cause:**
HPA not configured; Pods saturated CPU.

**Resolution:**
Implemented HPA based on CPU metrics.

---

### Incident 3: LoadBalancer Not Created

**Root Cause:**
IAM role missing ELB permissions.

**Resolution:**
Attached correct IAM policy to EKS worker nodes.

---

## Architect-Level Closing Statement

This task demonstrates core Kubernetes concepts and maps cleanly to AWS EKS production architectures using LoadBalancers, auto scaling, and fault-tolerant design principles.

