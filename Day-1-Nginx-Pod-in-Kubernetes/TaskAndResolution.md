
## Task Title

Create an Nginx Pod in Kubernetes

---

## Task Description

The DevOps team is onboarding Kubernetes for application management. The task is to create a Kubernetes **Pod** with the following requirements:

* Pod name: `pod-nginx`
* Container image: `nginx:latest`
* Container name: `nginx-container`
* Label: `app=nginx_app`
* Kubernetes access is already configured on `jump_host` using `kubectl`

---

## Resolution Steps

### Step 1: Create Pod Manifest File

**Command:**

```bash
vi pod-nginx.yaml
```

**Explanation:**
Creates a YAML manifest file where the Kubernetes Pod definition will be written. No `sudo` is required because the file is created in the user home directory.

---

### Step 2: Add Pod Configuration

**File Content (`pod-nginx.yaml`):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx_app
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

**Explanation:**

* `apiVersion: v1` → Core Kubernetes API for Pods
* `kind: Pod` → Defines a Pod resource
* `metadata.name` → Sets the Pod name
* `metadata.labels` → Adds labels for identification and selection
* `spec.containers` → Defines container specifications
* `name` → Container name inside the Pod
* `image` → Docker image with explicit `latest` tag

---

### Step 3: Verify YAML File Content

**Command:**

```bash
cat pod-nginx.yaml
```

**Output:**

```text
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: nginx_app
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
```

**Explanation:**
Displays the contents of the YAML file to ensure correctness before applying it to the cluster.

---

### Step 4: Create the Pod in Kubernetes

**Command:**

```bash
kubectl apply -f pod-nginx.yaml
```

**Output:**

```text
pod/pod-nginx created
```

**Explanation:**
Applies the YAML manifest to the Kubernetes cluster. Kubernetes API server processes the request and creates the Pod.

---

### Step 5: Verify Pod Status

**Command:**

```bash
kubectl get pods
```

**Output:**

```text
NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          20s
```

**Explanation:**
Confirms that the Pod is created successfully and is in the `Running` state.

---

### Step 6: Verify Pod Labels

**Command:**

```bash
kubectl get pod pod-nginx --show-labels
```

**Output:**

```text
NAME        READY   STATUS    RESTARTS   AGE   LABELS
pod-nginx   1/1     Running   0          77s   app=nginx_app
```

**Explanation:**
Displays the Pod along with its labels to confirm that `app=nginx_app` has been applied correctly.

---

## Final Status

* Pod Name: `pod-nginx` → ✅ Created
* Container Name: `nginx-container` → ✅ Correct
* Image: `nginx:latest` → ✅ Correct
* Label: `app=nginx_app` → ✅ Verified
* Pod Status: `Running` → ✅ Healthy

---

## Conclusion

The Kubernetes Pod was created successfully as per the task requirements. All validations confirm that the Pod is running with the correct image, container name, and labels.

---

## Interview Notes

* Pods are the smallest deployable unit in Kubernetes.
* YAML manifests are preferred for declarative and repeatable deployments.
* Labels are critical for service discovery and workload management.
* `kubectl apply` follows declarative configuration management best practices.

