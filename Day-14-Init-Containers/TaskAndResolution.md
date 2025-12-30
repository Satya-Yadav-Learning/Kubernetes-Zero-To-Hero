# TaskAndResolution.md

## Task Title

Implementation and Validation of Kubernetes Init Containers with Shared emptyDir Volume

---

## Task Objective

Design, deploy, and validate a Kubernetes Deployment using Init Containers to initialize shared state before application startup, and verify runtime behavior through logs and pod inspection.

---

## Environment Details

* Platform: Kubernetes (single-node lab / control-plane node)
* Namespace: default
* Access Node: host
* User: <redacted>
* Container Runtime: containerd

---

## Task Requirements

1. Create a Deployment named `ic-deploy-datacenter`
2. Replicas: `1`
3. Labels:

   * Deployment selector: `app=ic-datacenter`
   * Pod template labels: `app=ic-datacenter`
4. Init Container:

   * Name: `ic-msg-datacenter`
   * Image: `ubuntu:latest`
   * Command:

     ```bash
     /bin/bash -c "echo Init Done - Welcome to the organization > /ic/blog"
     ```
   * Volume mount:

     * Name: `ic-volume-datacenter`
     * Mount path: `/ic`
5. Main Container:

   * Name: `ic-main-datacenter`
   * Image: `ubuntu:latest`
   * Command:

     ```bash
     /bin/bash -c "while true; do cat /ic/blog; sleep 5; done"
     ```
   * Volume mount:

     * Name: `ic-volume-datacenter`
     * Mount path: `/ic`
6. Volume:

   * Name: `ic-volume-datacenter`
   * Type: `emptyDir`

---

## Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-datacenter
  template:
    metadata:
      labels:
        app: ic-datacenter
    spec:
      initContainers:
        - name: ic-msg-datacenter
          image: ubuntu:latest
          command:
            - /bin/bash
            - -c
            - echo Init Done - Welcome to the organization > /ic/blog
          volumeMounts:
            - name: ic-volume-datacenter
              mountPath: /ic
      containers:
        - name: ic-main-datacenter
          image: ubuntu:latest
          command:
            - /bin/bash
            - -c
            - while true; do cat /ic/blog; sleep 5; done
          volumeMounts:
            - name: ic-volume-datacenter
              mountPath: /ic
      volumes:
        - name: ic-volume-datacenter
          emptyDir: {}
```

---

## Execution Steps (With Full Command Output & Explanation)

### 1. Apply the Deployment Manifest

```bash
kubectl apply -f ic-deploy-datacenter.yml
```

**Output:**

```text
deployment.apps/ic-deploy-datacenter created
```

**Explanation:**

* `kubectl apply` submits the YAML manifest to the Kubernetes API server.
* Kubernetes validates the manifest and creates a Deployment object.
* The Deployment controller then creates a ReplicaSet and Pod.

---

### 2. Verify Deployment Status

```bash
kubectl get deployment ic-deploy-datacenter
```

**Output:**

```text
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-datacenter   1/1     1            1           40s
```

**Explanation:**

* `READY 1/1` → Desired replicas are running and ready.
* `UP-TO-DATE 1` → Latest Pod template is in use.
* `AVAILABLE 1` → Pod passed readiness checks and can serve traffic.

---

### 3. Verify Pod Creation Using Labels

```bash
kubectl get pods -l app=ic-datacenter
```

**Output:**

```text
NAME                                    READY   STATUS    RESTARTS   AGE
ic-deploy-datacenter-6cfdfb5fbf-c2vj8   1/1     Running   0          55s
```

**Explanation:**

* `-l app=ic-datacenter` filters Pods using labels.
* Confirms that the Deployment selector and Pod labels are aligned.
* `STATUS Running` indicates Init Containers completed successfully.

---

### 4. Describe the Pod (Init Container Validation)

```bash
kubectl describe pod ic-deploy-datacenter-6cfdfb5fbf-c2vj8
```

**Relevant Output (Init Container):**

```text
Init Containers:
  ic-msg-datacenter:
    State:      Terminated
    Reason:     Completed
    Exit Code:  0
```

**Explanation:**

* Shows Init Container lifecycle details.
* `Exit Code 0` confirms successful execution.
* Pod initialization is unblocked.

---

### 5. Fetch Logs from Main Container

```bash
kubectl logs ic-deploy-datacenter-6cfdfb5fbf-c2vj8 -c ic-main-datacenter
```

**Output (sample):**

```text
Init Done - Welcome to the organization
Init Done - Welcome to the organization
Init Done - Welcome to the organization
```

**Explanation:**

* Logs are retrieved from the main container.
* Confirms shared `emptyDir` volume usage.
* Verifies that Init Container data is consumed by the main container.

---

### 6. Attempting Logs Using Deployment Name (Common Mistake)

```bash
kubectl logs ic-deploy-datacenter -c ic-main-datacenter
```

**Output:**

```text
error: error from server (NotFound): pods "ic-deploy-datacenter" not found
```

**Explanation:**

* `kubectl logs` works only with Pod names, not Deployment names.
* Deployments are controllers, not runtime objects.

---

## Observations & Validation

### Init Container

* State: Terminated
* Reason: Completed
* Exit Code: 0
* Behavior: Writes initialization message to `/ic/blog`

### Main Container

* State: Running
* Behavior: Continuously reads and prints `/ic/blog` every 5 seconds

### Volume Behavior

* `emptyDir` shared between Init and Main containers
* Data persists for the Pod lifetime

---

## Common Errors Observed & Resolution

### Issue 1: `kubectl logs ic-deploy-datacenter` NotFound

**Cause:** Logs were requested using Deployment name instead of Pod name.

**Resolution:**

```bash
kubectl logs <pod-name> -c <container-name>
```

---

### Issue 2: Init Container Logs Empty

**Cause:** Output was redirected to a file instead of STDOUT.

**Resolution:**

* Expected behavior
* Validation done via main container logs

---

## Architecture Flow (Textual Diagram)

```
Pod Scheduled
    ↓
Init Container Runs
    ↓
Writes /ic/blog
    ↓
Init Container Exits (0)
    ↓
Main Container Starts
    ↓
Reads /ic/blog every 5s
```

---

# Scenario-Based Interview Questions & Answers

---

## L1 – Junior Engineer

### Q1. What is the purpose of an Init Container?

**Answer:**
Init Containers run before application containers to perform setup tasks required for the application to start.

### Q2. Why did the Pod not start until Init Container completed?

**Answer:**
Kubernetes enforces Init Containers as blocking steps; failure prevents Pod startup.

---

## L2 – Mid-Level Engineer

### Q1. Why was `emptyDir` used here?

**Answer:**
To share temporary data between Init and Main containers within the same Pod.

### Q2. Why are Init Container logs empty?

**Answer:**
Because output was redirected to a file instead of STDOUT.

---

## L3 – Senior Engineer

### Q1. What risks exist if Init Containers are slow or fail?

**Answer:**

* Deployment rollout stalls
* Pods remain in Init state
* Production availability may degrade

### Q2. How would you improve observability?

**Answer:**

* Log to STDOUT using `tee`
* Add timeouts and explicit error handling

---

## L4 – Architect Level

### Q1. When should Init Containers NOT be used?

**Answer:**
For long-running tasks, migrations, or background jobs; use Jobs or CI/CD pipelines instead.

### Q2. What design principle do Init Containers enforce?

**Answer:**
Separation of concerns and deterministic startup.

---

# AWS Scenario Mapping & Real Incident RCAs

---

## AWS Scenario 1: EKS Pods Stuck in Init State

**Incident:**
Pods stuck in `Init:CrashLoopBackOff`

**Root Cause:**
Init Container waiting for RDS connectivity; Security Group missing inbound rule

**Resolution:**
Updated SG rules; Pods recovered automatically

---

## AWS Scenario 2: Deployment Rollout Failure

**Incident:**
RollingUpdate stalled during peak traffic

**Root Cause:**
Init Container accessing S3 without proper IAM permissions (IRSA misconfigured)

**Resolution:**
IAM policy fixed; rollout resumed

---

## AWS Scenario 3: Full Outage During Blue-Green Deployment

**Incident:**
Database outage during deployment

**Root Cause:**
Init Container performed DB migration per Pod

**Correct Design:**
Use Kubernetes Job or pipeline-driven migration

---

## Final Takeaways

* Init Containers are startup gatekeepers
* Failures block Pods entirely
* Most production issues relate to IAM, networking, or storage
* Misuse of Init Containers is an architectural anti-pattern

---

**Document Status:** Interview-ready, production-aligned, and audit-grade

