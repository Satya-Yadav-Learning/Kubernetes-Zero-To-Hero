# TaskAndResolution.md

## Task Title

Troubleshooting and Fixing a Python Flask Application Deployment on Kubernetes

---

## Task Objective

Identify and fix misconfigurations in an existing Kubernetes Deployment and Service so that a Python Flask application becomes accessible via a specified NodePort.

---

## Environment Details

* Kubernetes Cluster: Preconfigured
* Namespace: default
* Access Node: host
* Tooling: kubectl
* Application Type: Python Flask

---

## Initial Problem Statement

* Deployment already exists: `python-deployment-devops`
* Service already exists: `python-service-devops`
* Application is not accessible
* Required NodePort: **32345**
* Flask default application port: **5000**

---

## Step 1: Check Pod Status

### Command

```bash
kubectl get pods
```

### Output

```text
NAME                                       READY   STATUS             RESTARTS   AGE
python-deployment-devops-678b746b7-vv6ph   0/1     ImagePullBackOff   0          73s
```

### Explanation

* Pod is not running.
* `ImagePullBackOff` indicates Kubernetes cannot pull the container image.
* Application cannot start until this is resolved.

---

## Step 2: Check Deployment Status

### Command

```bash
kubectl get deployment python-deployment-devops
```

### Output

```text
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
python-deployment-devops   0/1     1            0           2m41s
```

### Explanation

* Deployment has created a Pod, but it is not available.
* Confirms issue is at Pod/container level.

---

## Step 3: Describe Deployment (Find Root Cause)

### Command

```bash
kubectl describe deployment python-deployment-devops
```

### Output (Relevant Section)

```text
Containers:
 python-container-devops:
  Image: poroko/flask-app-demo
  Port: 5000/TCP
```

### Explanation

* Image name is **incorrect**.
* The task requires image: `poroko/flask-demo-appimage`.
* Because the image does not exist, Kubernetes fails to pull it.

---

## Step 4: Inspect Service Configuration

### Command

```bash
kubectl get svc
```

### Output

```text
python-service-devops   NodePort   10.96.203.218   <none>   8080:32345/TCP
```

---

### Describe the Service

```bash
kubectl describe svc python-service-devops
```

### Output (Relevant Section)

```text
TargetPort: 8080/TCP
NodePort:   32345/TCP
Endpoints:  <none>
```

### Explanation

* Flask app listens on port **5000**.
* Service forwards traffic to **8080**, which is wrong.
* No endpoints because Pod is not running.

---

## Step 5: Fix Deployment Image

### Command

```bash
kubectl edit deployment python-deployment-devops
```

### Change Applied

```yaml
image: poroko/flask-demo-appimage
```

### Output

```text
deployment.apps/python-deployment-devops edited
```

### Explanation

* Correct image allows Kubernetes to pull and start the container.
* Deployment automatically creates a new Pod.

---

## Step 6: Fix Service targetPort

### Command

```bash
kubectl edit svc python-service-devops
```

### Change Applied

```yaml
ports:
- protocol: TCP
  port: 8080
  targetPort: 5000
  nodePort: 32345
```

### Output

```text
service/python-service-devops edited
```

### Explanation

* `targetPort: 5000` correctly maps traffic to Flask.
* `nodePort: 32345` exposes the service externally.

---

## Step 7: Verify Pod is Running

### Command

```bash
kubectl get pods
```

### Output

```text
python-deployment-devops-7859694dcf-fb7lk   1/1   Running   0   96s
```

### Explanation

* Pod successfully pulled image and started.
* Application is now running.

---

## Step 8: Verify Application Logs

### Command

```bash
kubectl logs python-deployment-devops-7859694dcf-fb7lk
```

### Output

```text
* Running on http://0.0.0.0:5000/
```

### Explanation

* Confirms Flask app is listening on port 5000.

---

## Step 9: Verify Service Endpoints

### Command

```bash
kubectl get endpoints python-service-devops
```

### Output

```text
python-service-devops   10.244.0.6:5000
```

### Explanation

* Service correctly routes traffic to the Pod on port 5000.

---

## Step 10: Access Application

### Command

```bash
kubectl get nodes -o wide
```

### Output

```text
INTERNAL-IP: 172.17.0.2
```

### Access URL

```text
http://172.17.0.2:32345
```

### Result

* Flask application loads successfully.

---

# Scenario-Based Interview Questions & Answers

---

## L1 – Junior Level

**Q: What does ImagePullBackOff mean?**
A: Kubernetes cannot pull the container image due to wrong name, tag, or registry access.

---

## L2 – Mid Level

**Q: Why must Service targetPort match container port?**
A: Service forwards traffic to the container’s listening port; mismatch causes connectivity failure.

---

## L3 – Senior Level

**Q: Why didn’t the Service show endpoints initially?**
A: Endpoints are created only when matching Pods are running and ready.

---

## L4 – Architect Level

**Q: How would you prevent such issues in production?**
A: Use CI validation, image scanning, health checks, and automated tests.

---

# AWS EKS – Scenario Mapping & Real Incident RCAs

---

## Incident 1: Production Outage Due to ImagePullBackOff

* **Cause:** Wrong image tag pushed to Deployment
* **Impact:** Application downtime
* **Fix:** Rolled back Deployment and enforced image tag validation

---

## Incident 2: NodePort Exposed but App Unreachable

* **Cause:** targetPort mismatch
* **Fix:** Corrected Service configuration and added readiness probes

---

## Incident 3: No Service Endpoints in EKS

* **Cause:** Pod crash due to bad image
* **Fix:** Implemented pre-deployment checks in CI/CD

---

## Final Takeaways

* Image correctness is critical
* Service targetPort must match container port
* NodePort exposes service but does not guarantee app health
* Systematic troubleshooting prevents guesswork

---

**Document Status:** Complete, interview-ready, and production-aligned

