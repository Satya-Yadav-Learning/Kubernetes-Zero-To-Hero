# Kubernetes Job — countdown-nautilus

## Objective

Create a Kubernetes Job with the following requirements:

| Requirement       | Configuration                  |
| ----------------- | ------------------------------ |
| Job name          | `countdown-nautilus`           |
| Pod template name | `countdown-nautilus`           |
| Container name    | `container-countdown-nautilus` |
| Image             | `ubuntu:latest`                |
| Restart policy    | `Never`                        |
| Command           | `sleep 5`                      |

---

# 1. Verify Kubernetes Cluster

Check that the Kubernetes node is available:

```bash
kubectl get nodes
```

Output:

```text
NAME        STATUS   ROLES           AGE   VERSION
jump-host   Ready    control-plane   37m   v1.34.1+k3s1
```

### Explanation

The node is:

* `Ready` → Kubernetes can schedule workloads on it.
* `control-plane` → The node is running Kubernetes control-plane components.
* Kubernetes version is `v1.34.1+k3s1`.

---

# 2. Create the Job Manifest

Create the YAML file:

```bash
vi countdown-nautilus.yaml
```

Use the following manifest:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-nautilus
spec:
  template:
    metadata:
      name: countdown-nautilus
    spec:
      containers:
        - name: container-countdown-nautilus
          image: ubuntu:latest
          command:
            - /bin/sh
            - -c
            - "sleep 5"
      restartPolicy: Never
```

### Explanation

### `apiVersion`

```yaml
apiVersion: batch/v1
```

The `batch/v1` API is used for Kubernetes batch workloads such as Jobs and CronJobs.

### `kind`

```yaml
kind: Job
```

Creates a Kubernetes Job.

A Job is designed to run a task to completion rather than continuously serving traffic.

### Job name

```yaml
metadata:
  name: countdown-nautilus
```

This is the Kubernetes Job's name.

### Pod template name

```yaml
template:
  metadata:
    name: countdown-nautilus
```

This defines the metadata name for the Pod template as required by the task.

### Container name

```yaml
containers:
  - name: container-countdown-nautilus
```

This is the name of the container inside the Pod.

### Image

```yaml
image: ubuntu:latest
```

The Job uses the Ubuntu image with the `latest` tag.

### Command

```yaml
command:
  - /bin/sh
  - -c
  - "sleep 5"
```

This executes:

```text
/bin/sh -c "sleep 5"
```

The container waits for 5 seconds and then exits successfully.

### Restart policy

```yaml
restartPolicy: Never
```

The Pod's container will not be restarted by the kubelet after it exits.

The Job controller is responsible for ensuring that the desired completion is achieved.

---

# 3. Validate the Manifest

Before creating the actual resource, perform a client-side dry run:

```bash
kubectl apply --dry-run=client -f countdown-nautilus.yaml
```

Output:

```text
job.batch/countdown-nautilus created (dry run)
```

### Explanation

This validates the manifest using the Kubernetes client without actually creating the Job.

This is a useful practice before applying configuration changes.

---

# 4. Create the Job

Apply the manifest:

```bash
kubectl apply -f countdown-nautilus.yaml
```

Output:

```text
job.batch/countdown-nautilus created
```

The Job has now been created in the Kubernetes cluster.

---

# 5. Verify the Job

Run:

```bash
kubectl get job countdown-nautilus
```

Output:

```text
NAME                 STATUS     COMPLETIONS   DURATION   AGE
countdown-nautilus   Complete   1/1           11s        75s
```

### Interpretation

`STATUS = Complete`

The Job successfully finished.

`COMPLETIONS = 1/1`

One successful completion was required and one successful completion was achieved.

The Job therefore completed successfully.

---

# 6. Find the Pod Created by the Job

Run:

```bash
kubectl get pods -l job-name=countdown-nautilus
```

Output:

```text
NAME                       READY   STATUS      RESTARTS   AGE
countdown-nautilus-bls2b   0/1     Completed   0          2m19s
```

The Job created the following Pod:

```text
countdown-nautilus-bls2b
```

### Explanation

The relationship is:

```text
Job
 └── Pod
      └── Container
           └── Ubuntu image
```

The Pod is `Completed`, meaning its container finished execution successfully.

`RESTARTS = 0` confirms that the container was not restarted.

---

# 7. Identify the Node

Run:

```bash
kubectl get pods -l job-name=countdown-nautilus -o wide
```

Output:

```text
NAME                       READY   STATUS      RESTARTS   AGE     IP          NODE        NOMINATED NODE   READINESS GATES
countdown-nautilus-bls2b   0/1     Completed   0          2m57s   10.22.0.9   jump-host   <none>           <none>
```

The Pod ran on:

```text
jump-host
```

### Explanation

The `NODE` column tells us which Kubernetes node hosted the Pod.

The Kubernetes scheduler selected `jump-host` because it was the available `Ready` node.

---

# 8. Verify Container Name

Run:

```bash
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.containers[*].name}'; echo
```

Output:

```text
container-countdown-nautilus
```

This confirms the required container name.

---

# 9. Verify Image

Run:

```bash
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.containers[*].image}'; echo
```

Output:

```text
ubuntu:latest
```

This confirms that the required Ubuntu image and `latest` tag are being used.

---

# 10. Verify Restart Policy

Run:

```bash
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.restartPolicy}'; echo
```

Output:

```text
Never
```

This confirms the required restart policy.

---

# 11. Verify Container Command

Run:

```bash
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.containers[0].command[*]}'; echo
```

Output:

```text
/bin/sh -c sleep 5
```

This confirms that the container executes the required `sleep 5` command.

---

# 12. Final Configuration Verification

The Job now satisfies all requirements:

```text
Job:
countdown-nautilus

Pod template:
countdown-nautilus

Container:
container-countdown-nautilus

Image:
ubuntu:latest

Command:
/bin/sh -c sleep 5

Restart policy:
Never

Job status:
Complete

Completion:
1/1

Pod:
countdown-nautilus-bls2b

Pod status:
Completed

Node:
jump-host
```

---

# Kubernetes Job Architecture

The execution flow is:

```text
Job
 |
 +-- Pod Template
      |
      +-- Pod
           |
           +-- Container
                |
                +-- ubuntu:latest
                     |
                     +-- /bin/sh -c "sleep 5"
```

The Kubernetes control flow is:

```text
kubectl apply
      |
      v
Kubernetes API Server
      |
      v
Job Controller
      |
      v
Pod created
      |
      v
Scheduler selects node
      |
      v
jump-host
      |
      v
Container starts
      |
      v
sleep 5
      |
      v
Container exits successfully
      |
      v
Pod = Completed
      |
      v
Job = Complete
```

---

# Job vs Pod

A common interview question is:

**What is the difference between a Job and a Pod?**

A Pod is the execution unit that runs one or more containers.

A Job is a controller that manages Pods and ensures that a specified task completes successfully.

For this task:

```text
Job:       countdown-nautilus
Pod:       countdown-nautilus-bls2b
Container: container-countdown-nautilus
```

The Job created the Pod, and the Pod ran the container.

---

# Why Use a Job?

Jobs are appropriate for tasks such as:

* Database migrations
* Batch processing
* Data processing
* One-time scripts
* Cleanup operations
* Backup operations
* ETL workloads
* Initialization tasks

For example:

```text
Database Migration Job
        |
        v
Run migration
        |
        v
Migration succeeds
        |
        v
Job Complete
```

---

# Job vs CronJob

## Job

A Job runs a task until successful completion.

Example:

```text
Create Job
   ↓
Run task
   ↓
Complete
```

## CronJob

A CronJob creates Jobs according to a schedule.

Example:

```text
CronJob
   |
   +-- Job 1
   |
   +-- Job 2
   |
   +-- Job 3
```

A CronJob is therefore useful for scheduled tasks such as:

```text
Every 10 minutes
Every hour
Every day
Every Sunday
```

---

# Important Kubernetes Commands

## List Jobs

```bash
kubectl get jobs
```

## Get a specific Job

```bash
kubectl get job countdown-nautilus
```

## Describe a Job

```bash
kubectl describe job countdown-nautilus
```

Useful for troubleshooting:

* Events
* Pod creation
* Completion status
* Failure information
* Backoff information

## List Pods

```bash
kubectl get pods
```

## Find Pods belonging to a Job

```bash
kubectl get pods -l job-name=countdown-nautilus
```

## Show Pod and Node

```bash
kubectl get pods -l job-name=countdown-nautilus -o wide
```

## View Pod details

```bash
kubectl describe pod countdown-nautilus-bls2b
```

## View container logs

```bash
kubectl logs countdown-nautilus-bls2b
```

For this particular Job, `sleep 5` does not produce stdout, so empty logs are normal.

---

# L1 Troubleshooting

## Job does not exist

Check:

```bash
kubectl get jobs
```

Then:

```bash
kubectl get job countdown-nautilus
```

If it does not exist, check whether the manifest was actually applied.

---

## Job is not completing

Check:

```bash
kubectl get job countdown-nautilus
```

Then:

```bash
kubectl get pods -l job-name=countdown-nautilus
```

If the Pod is failing:

```bash
kubectl describe pod <pod-name>
```

Check the Events section.

---

## Pod is stuck in Pending

Run:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Events
```

Common causes include:

* Insufficient resources
* Node unavailable
* Scheduling constraints
* Image pull problems
* Taints/tolerations

Then check:

```bash
kubectl get nodes
```

---

# L2 Troubleshooting

For a Job that repeatedly fails:

```bash
kubectl describe job countdown-nautilus
```

Then:

```bash
kubectl get pods -l job-name=countdown-nautilus
```

Then inspect the failed Pod:

```bash
kubectl describe pod <pod-name>
```

And logs:

```bash
kubectl logs <pod-name>
```

Typical investigation flow:

```text
Job
 ↓
Pod
 ↓
Container
 ↓
Logs
 ↓
Events
```

---

# L3 Troubleshooting

For advanced investigation, check the complete Job specification:

```bash
kubectl get job countdown-nautilus -o yaml
```

Check the Pod specification:

```bash
kubectl get pod <pod-name> -o yaml
```

Check the node:

```bash
kubectl get nodes
```

Check node details:

```bash
kubectl describe node jump-host
```

Look for:

* CPU/memory pressure
* Disk pressure
* Network problems
* Node conditions
* Scheduling issues
* Container runtime problems

---

# L4 Troubleshooting

At the platform level, investigate:

```text
Kubernetes API Server
        ↓
Controller Manager
        ↓
Scheduler
        ↓
Kubelet
        ↓
Container Runtime
        ↓
Container
```

For a production incident, correlate:

* Kubernetes Events
* Pod status
* Container logs
* Node conditions
* Kubelet logs
* Container runtime logs
* Resource utilization
* Network telemetry
* Application monitoring

The goal is to identify whether the failure is:

```text
Application
     OR
Container
     OR
Pod
     OR
Node
     OR
Kubernetes Control Plane
     OR
Infrastructure
```

---

# Production Best Practices

## 1. Avoid unnecessary `latest`

Although this task explicitly requires:

```yaml
image: ubuntu:latest
```

production workloads should generally use a controlled image version or digest.

Example:

```yaml
image: ubuntu:24.04
```

or preferably a verified image digest.

This improves reproducibility.

---

## 2. Define resource requests and limits

Production Jobs should generally define CPU and memory requirements.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

This helps Kubernetes schedule workloads predictably and protects cluster resources.

---

## 3. Configure retry behavior

Jobs can use:

```yaml
backoffLimit: 3
```

This controls how many retries the Job controller permits after failures.

---

## 4. Make Jobs idempotent

A production Job should ideally be safe to retry.

For example, a database migration should not corrupt data if Kubernetes executes it again after a failure.

---

## 5. Set appropriate timeouts

Long-running Jobs should be designed with appropriate execution limits.

A stuck Job should not consume cluster resources indefinitely.

---

# AWS Mapping

The same Kubernetes concepts can appear in AWS environments running Kubernetes, such as Amazon EKS.

| Kubernetes | AWS/EKS                                       |
| ---------- | --------------------------------------------- |
| Job        | Kubernetes Job running in EKS                 |
| Pod        | EKS Pod                                       |
| Node       | EC2-backed worker node or managed node        |
| Container  | Container running through container runtime   |
| Image      | Container image from ECR or another registry  |
| Logs       | CloudWatch / observability platform           |
| Metrics    | CloudWatch / Prometheus / monitoring platform |

Typical architecture:

```text
Developer
    |
    v
kubectl
    |
    v
EKS API Server
    |
    v
Job Controller
    |
    v
Pod
    |
    v
EKS Worker Node
    |
    v
Container
```

---

# Interview Questions and Answers

## Q1. What is a Kubernetes Job?

A Kubernetes Job is a workload controller that creates Pods and ensures that a specified task runs to successful completion.

---

## Q2. What is the difference between Job and Deployment?

A Deployment is designed for continuously running applications.

A Job is designed for finite tasks that eventually complete.

Example:

```text
Deployment → Web application
Job        → Database migration
```

---

## Q3. Why is `restartPolicy: Never` used?

It prevents the kubelet from restarting the container inside the same Pod after it exits.

For a Job, the Job controller can create another Pod when necessary according to the Job's retry behavior.

---

## Q4. What happens after `sleep 5` finishes?

The shell process exits successfully.

Then:

```text
Container → Terminated
Pod       → Completed
Job       → Complete
```

---

## Q5. Why is the Pod showing `0/1` but `Completed`?

`0/1` means zero containers are currently ready.

That is normal because the container has already finished.

`Completed` means the container terminated successfully.

---

## Q6. How do you find which Pod belongs to a Job?

Use the Job label:

```bash
kubectl get pods -l job-name=countdown-nautilus
```

---

## Q7. How do you find which node is running a Pod?

Use:

```bash
kubectl get pods -o wide
```

The `NODE` column shows the node.

For a specific Pod:

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.nodeName}'; echo
```

---

## Q8. How do you find the container name?

Use:

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'; echo
```

---

## Q9. How do you find the image used by a container?

Use:

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].image}'; echo
```

---

## Q10. How would you troubleshoot a failed Job?

I would follow this sequence:

```text
kubectl get job
        ↓
kubectl describe job
        ↓
kubectl get pods
        ↓
kubectl describe pod
        ↓
kubectl logs
        ↓
Check Events
        ↓
Check Node
        ↓
Check Kubernetes/infrastructure layer
```

This helps isolate whether the issue is with the Job specification, Pod, container, image, node, or cluster.

---

# Scenario-Based Interview Questions

## Scenario 1: Job is stuck in Pending

**Interviewer:** A Job was created but its Pod remains Pending. What do you check?

**Answer:**

First I would check:

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <pod-name>
```

I would inspect Events for scheduling errors.

Then:

```bash
kubectl get nodes
```

and:

```bash
kubectl describe node <node-name>
```

I would investigate resource availability, taints, tolerations, affinity rules, node conditions, and scheduling constraints.

---

# Scenario 2: Job is failing repeatedly

**Interviewer:** The Job keeps creating failed Pods. How do you troubleshoot?

**Answer:**

I would first check:

```bash
kubectl get job
```

Then:

```bash
kubectl describe job <job-name>
```

Then identify the failed Pod:

```bash
kubectl get pods -l job-name=<job-name>
```

Then inspect:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

I would determine whether the failure is caused by the command, image, permissions, configuration, dependency, resource limitation, or external service.

---

# Scenario 3: Container exits immediately

**Interviewer:** A Job Pod starts and immediately becomes Completed. Is that necessarily a problem?

**Answer:**

No.

A Job is expected to terminate after completing its task.

For example:

```bash
sleep 5
```

runs for five seconds and exits.

If the exit code is `0`, the Pod becomes:

```text
Completed
```

and the Job becomes:

```text
Complete
```

That is the expected behavior.

---

# Scenario 4: Container exits with error

If the command exits with a non-zero exit code, the Job considers that attempt unsuccessful.

I would inspect:

```bash
kubectl logs <pod-name>
```

and:

```bash
kubectl describe pod <pod-name>
```

Then check the Job:

```bash
kubectl describe job <job-name>
```

I would investigate the root cause before increasing retries.

---

# Key Interview Concept: Job Lifecycle

Remember this sequence:

```text
Job Created
     ↓
Job Controller
     ↓
Pod Created
     ↓
Scheduler
     ↓
Node Selected
     ↓
Container Started
     ↓
Command Executed
     ↓
Container Exits
     ↓
Pod Completed
     ↓
Job Complete
```

For this task:

```text
countdown-nautilus
        ↓
countdown-nautilus-bls2b
        ↓
container-countdown-nautilus
        ↓
ubuntu:latest
        ↓
sleep 5
        ↓
Completed
        ↓
1/1 Complete
```

---

# Important Commands to Remember

```bash
# Check nodes
kubectl get nodes

# Create Job
kubectl apply -f countdown-nautilus.yaml

# Validate manifest without creating
kubectl apply --dry-run=client -f countdown-nautilus.yaml

# Check Job
kubectl get job countdown-nautilus

# Check all Jobs
kubectl get jobs

# Find Job Pods
kubectl get pods -l job-name=countdown-nautilus

# Show Pod + Node
kubectl get pods -l job-name=countdown-nautilus -o wide

# Describe Job
kubectl describe job countdown-nautilus

# Describe Pod
kubectl describe pod countdown-nautilus-bls2b

# Check container name
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.containers[*].name}'; echo

# Check image
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.containers[*].image}'; echo

# Check restart policy
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.restartPolicy}'; echo

# Check command
kubectl get pod countdown-nautilus-bls2b -o jsonpath='{.spec.containers[0].command[*]}'; echo

# Check logs
kubectl logs countdown-nautilus-bls2b
```

---

# Final Result

The Kubernetes Job `countdown-nautilus` was successfully created and completed.

Verified configuration:

```text
Job Name:
countdown-nautilus

Template Name:
countdown-nautilus

Container Name:
container-countdown-nautilus

Image:
ubuntu:latest

Command:
/bin/sh -c sleep 5

Restart Policy:
Never

Job Status:
Complete

Completions:
1/1

Pod:
countdown-nautilus-bls2b

Pod Status:
Completed

Node:
jump-host
```

**Task Status: SUCCESSFUL**

