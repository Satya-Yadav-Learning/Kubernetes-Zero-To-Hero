# TaskAndResolution.md

## Task

Configure and test a Kubernetes workload that prints a greeting message using environment variables. The workload should execute once, avoid restart loops, and allow output verification via logs.

All steps below were executed from the **host**.

---

## Requirements

* Create a Pod named `print-envars-greeting`
* Container name: `print-env-container`
* Image: `bash`
* Environment variables:

  * `GREETING=Welcome to`
  * `COMPANY=natural`
  * `GROUP=Industries`
* Command (must be exact):

  ```bash
  ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
  ```
* `restartPolicy: Never`
* Verify output using `kubectl logs`

---

## Step 1: Create Pod Manifest

```bash
host ~$ vi print-envars-greeting.yaml
```

### print-envars-greeting.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
    - name: print-env-container
      image: bash
      command:
        - /bin/sh
        - -c
        - echo "$(GREETING) $(COMPANY) $(GROUP)"
      env:
        - name: GREETING
          value: "Welcome to"
        - name: COMPANY
          value: "natural"
        - name: GROUP
          value: "Industries"
```

### Explanation

* Environment variables are injected at runtime
* `/bin/sh -c` enables shell variable expansion
* The container runs once and exits

---

## Step 2: Apply the Pod

```bash
host ~$ kubectl apply -f print-envars-greeting.yaml
```

### Output

```text
pod/print-envars-greeting created
```

---

## Step 3: Verify Pod Status

```bash
host ~$ kubectl get pod print-envars-greeting
```

### Output

```text
NAME                    READY   STATUS      RESTARTS   AGE
print-envars-greeting   0/1     Completed   0          56s
```

### Explanation

* `Completed` indicates successful one-time execution
* `restartPolicy: Never` prevents restarts

---

## Step 4: Verify Output

```bash
host ~$ kubectl logs -f print-envars-greeting
```

### Output

```text
Welcome to natural Industries
```

### Confirmation

* Environment variables resolved correctly
* Command executed exactly as required

---

## Scenario-Based Interview Questions & Answers

### L1 – Junior DevOps (Fundamentals)

**Q1. Why does the Pod show `Completed` status?**
Because the container executed its command successfully and exited normally.

**Q2. Why is `restartPolicy: Never` used?**
To prevent Kubernetes from restarting the Pod after completion.

---

### L2 – Mid-Level DevOps (Operations)

**Q3. How are environment variables passed into the container?**
They are injected by the kubelet at container startup and accessible at runtime.

**Q4. Why is `/bin/sh -c` required?**
It allows shell-based variable expansion using `$(VAR_NAME)` syntax.

---

### L3 – Senior DevOps (Reliability & Design)

**Q5. What would happen if restartPolicy was set to `Always`?**
The Pod would enter a CrashLoopBackOff state after completing execution.

**Q6. When should a Job be used instead of a Pod?**
When execution reliability, retries, or completion guarantees are required.

---

### L4 – Architect Level (System Design)

**Q7. How would you productionize this pattern?**
By converting it to a Kubernetes Job with retry logic and observability.

---

## Conversion: Pod → Kubernetes Job

### Why Use a Job?

* Guarantees execution completion
* Supports retries on failure
* Suitable for batch and one-time workloads

### Kubernetes Job Manifest

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: print-envars-greeting-job
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: print-env-container
          image: bash
          command:
            - /bin/sh
            - -c
            - echo "$(GREETING) $(COMPANY) $(GROUP)"
          env:
            - name: GREETING
              value: "Welcome to"
            - name: COMPANY
              value: "natural"
            - name: GROUP
              value: "Industries"
```

---

## AWS EKS Mapping

### Kubernetes → AWS EKS Mapping

| Kubernetes Concept | AWS Equivalent                       |
| ------------------ | ------------------------------------ |
| Pod                | Container running on EC2 worker node |
| Job                | Batch job on EKS                     |
| restartPolicy      | Job retry strategy                   |

---

## AWS Scenario-Based Q&A

**Q8. How is a Kubernetes Job executed in EKS?**
EKS schedules the Job Pod onto EC2 worker nodes, and execution is managed by the Job controller.

**Q9. How do you monitor Job execution in AWS?**
Using CloudWatch Container Insights and Kubernetes events.

---

## InitContainer Mapping (AWS & Kubernetes)

### When to Use InitContainers

* Pre-task initialization
* Configuration generation
* Dependency checks

### Example InitContainer Use Case

* InitContainer prints greeting
* Main container starts application

---

## Final Summary

This task demonstrates a one-time execution pattern using Kubernetes environment variables, validates correct Pod lifecycle behavior, and shows how the same design scales to Kubernetes Jobs and AWS EKS production workloads.

