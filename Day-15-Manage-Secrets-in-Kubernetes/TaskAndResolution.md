# TaskAndResolution.md

## Task Title

Securely Managing License Information Using Kubernetes Secrets

---

## Task Objective

Store sensitive license/password information securely in a Kubernetes cluster using **Kubernetes Secrets**, mount the secret into a Pod as a volume, and verify consumption inside a running container.

---

## Environment Details

* Platform: Kubernetes Cluster
* Namespace: default
* Access Node: host
* Container Runtime: containerd

---

## Task Requirements

1. A secret key file `beta.txt` is already present at `/opt/beta.txt` on the host
2. Create a **generic Kubernetes Secret** named `beta` using the contents of `beta.txt`
3. Create a Pod named `secret-datacenter`
4. Pod container configuration:

   * Container name: `secret-container-datacenter`
   * Image: `fedora:latest`
   * Command: `sleep` (to keep container running)
5. Mount the created secret inside the container at path `/opt/games`
6. Verify the secret content inside the running container

---

## Step-by-Step Execution With Commands, Output & Explanation

### Step 1: Verify the Secret Source File

```bash
cd /opt
ls -la
cat beta.txt
```

**Output:**

```text
-rw-r--r-- 1 root root 7 Dec 30 05:20 beta.txt
5ecur3
```

**Explanation:**

* Confirms that the secret source file exists
* File contains sensitive license/password data

---

### Step 2: Create the Kubernetes Secret

```bash
kubectl create secret generic beta --from-file=/opt/beta.txt
```

**Output:**

```text
secret/beta created
```

**Explanation:**

* `generic` creates an Opaque secret
* Kubernetes Base64-encodes file content automatically
* Secret is stored securely in etcd

---

### Step 3: Validate the Secret

```bash
kubectl describe secret beta
```

**Output:**

```text
Name:         beta
Type:         Opaque
Data:
  beta.txt: 7 bytes
```

**Explanation:**

* Confirms secret creation
* Actual value is hidden for security reasons

---

### Step 4: Create the Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
spec:
  containers:
    - name: secret-container-datacenter
      image: fedora:latest
      command:
        - sleep
        - "3600"
      volumeMounts:
        - name: beta-volume
          mountPath: /opt/games
          readOnly: true
  volumes:
    - name: beta-volume
      secret:
        secretName: beta
```

---

### Step 5: Deploy the Pod

```bash
kubectl apply -f secret-datacenter.yml
```

**Output:**

```text
pod/secret-datacenter created
```

---

### Step 6: Verify Pod Status

```bash
kubectl get pod secret-datacenter
```

**Output:**

```text
NAME                READY   STATUS    RESTARTS   AGE
secret-datacenter   1/1     Running   0          <time>
```

**Explanation:**

* Pod is running successfully
* Container kept alive using `sleep`

---

### Step 7: Verify Secret Inside the Container

```bash
kubectl exec -it secret-datacenter -- /bin/bash
ls /opt/games
cat /opt/games/beta.txt
```

**Output:**

```text
beta.txt
5ecur3
```

**Explanation:**

* Each secret key is exposed as a file
* Data is decoded at runtime
* Secret mounted as read-only volume

---

## Architecture Flow

```
[ beta.txt on host ]
        ↓
[ Kubernetes Secret ]
        ↓
[ Pod Volume Mount ]
        ↓
[ /opt/games/beta.txt inside container ]
```

---

# Scenario-Based Interview Questions & Answers

---

## L1 – Junior Level (Fundamentals)

**Q1. What is a Kubernetes Secret?**
A Kubernetes Secret stores sensitive data such as passwords, tokens, or license keys securely.

**Q2. How is secret data stored in Kubernetes?**
Secret values are Base64-encoded and stored in etcd.

---

## L2 – Mid-Level (Usage & Debugging)

**Q1. How can a secret be consumed inside a Pod?**
As environment variables or mounted as files using volumes.

**Q2. Why is volume-based secret mounting preferred?**
It reduces exposure risk compared to environment variables.

---

## L3 – Senior Level (Design & Security)

**Q1. What are common reasons Pods fail when using Secrets?**
Missing secrets, wrong key names, namespace mismatch, or RBAC restrictions.

**Q2. How do you rotate secrets without downtime?**
Update the secret and restart Pods or use external secret controllers.

---

## L4 – Architect Level (Production & Strategy)

**Q1. Why are native Kubernetes Secrets not sufficient for enterprise security?**
They rely on etcd security and require additional controls for compliance.

**Q2. What is the recommended approach for managing secrets in production?**
Integrate Kubernetes with external secret managers.

---

# AWS EKS – Real Production Incident RCAs

---

## Incident 1: Pods CrashLooping Due to Missing Secret

**Root Cause:**
Secret was deleted manually while Pods were running

**Impact:**
Application outage due to missing credentials

**Resolution:**
Recreated secret and restarted Pods

---

## Incident 2: Security Exposure via Environment Variables

**Root Cause:**
Secrets injected as env vars were exposed via debug logs

**Resolution:**
Migrated to volume-mounted secrets

---

## Incident 3: Compliance Failure (Audit Finding)

**Root Cause:**
Secrets stored in etcd without encryption at rest

**Resolution:**
Enabled etcd encryption and moved to AWS Secrets Manager with IRSA

---

## Key Takeaways

* Kubernetes Secrets protect sensitive data
* Base64 encoding is not encryption
* Volume mounts are safer than env vars
* Production clusters should use external secret managers

---

**Document Status:** Sanitized, interview-ready, production-aligned

