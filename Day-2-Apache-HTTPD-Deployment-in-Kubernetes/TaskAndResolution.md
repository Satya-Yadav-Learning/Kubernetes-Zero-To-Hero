## Task Title

Create an Apache HTTPD Deployment in Kubernetes

---

## Task Description

Kubernetes Deployment for the Apache HTTP Server using the `httpd:latest` image.

---

## Environment Verification (Pre-Checks)

### Check kubectl Client Installation

**Command:**

```bash
kubectl version --client
```

**Output:**

```text
Client Version: v1.30.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

**Explanation:**
Confirms that the `kubectl` CLI is installed & available for use.

---

### Check Cluster Connectivity

**Command:**

```bash
kubectl cluster-info
```

**Output:**

```text
Kubernetes control plane is running at https://notecloud-control-plane:6443
CoreDNS is running at https://notecloud-control-plane:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

**Explanation:**
Verifies that `kubectl` can successfully communicate with the Kubernetes control plane.

---

### Verify Current Context

**Command:**

```bash
kubectl config current-context
```

**Output:**

```text
kind-notecloud
```

**Explanation:**
Shows the active Kubernetes context, confirming which cluster `kubectl` is currently pointing to.

---

### List Available Contexts

**Command:**

```bash
kubectl config get-contexts
```

**Output:**

```text
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         kind-notecloud   kind-notecloud   kind-notecloud
```

**Explanation:**
Displays all configured contexts and highlights the active one.

---

### Verify Node Status

**Command:**

```bash
kubectl get nodes
```

**Output:**

```text
NAME                      STATUS   ROLES           AGE   VERSION
notecloud-control-plane   Ready    control-plane   23m   v1.27.16-1+f5da3b717fc217
```

**Explanation:**
Confirms that the Kubernetes node is in `Ready` state and able to schedule workloads.

---

## Deployment Creation (Declarative Method – Recommended)

### Step 1: Create Deployment Manifest

**Command:**

```bash
vi httpd-deployment.yaml
```

**Explanation:**
Creates a YAML file that will hold the Deployment definition. Editing files in the home directory does not require `sudo`.

---

### Step 2: Add Deployment Configuration

**File Content (`httpd-deployment.yaml`):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
        - name: httpd-container
          image: httpd:latest
```

**Explanation:**

* `apiVersion: apps/v1` → Required API for Deployments
* `kind: Deployment` → Ensures controller-managed Pods
* `metadata.name` → Name of the Deployment
* `replicas: 1` → Runs one Pod instance
* `selector.matchLabels` → Links Deployment to its Pods
* `template` → Pod template managed by Deployment
* `containers.name` → Container name
* `image` → Apache HTTPD image with explicit tag

---

### Step 3: Apply Deployment Manifest

**Command:**

```bash
kubectl apply -f httpd-deployment.yaml
```

**Output:**

```text
deployment.apps/httpd created
```

**Explanation:**
Submits the desired state defined in the YAML file to the Kubernetes API server. Kubernetes creates the Deployment and associated ReplicaSet and Pod.

---

### Step 4: Verify Deployment Status

**Command:**

```bash
kubectl get deployment
```

**Output:**

```text
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
httpd   1/1     1            1           33s
```

**Explanation:**
Confirms that the Deployment has the desired number of replicas available and up to date.

---

### Step 5: Verify Pod Creation

**Command:**

```bash
kubectl get pods
```

**Output:**

```text
NAME                    READY   STATUS    RESTARTS   AGE
httpd-8fd98cfbb-975gc   1/1     Running   0          50s
```

**Explanation:**
Validates that the Pod created by the Deployment is running successfully.

---

## Alternative Method (Imperative – Not Recommended for Production)

### Create Deployment Using kubectl Command

**Command:**

```bash
kubectl create deployment httpd --image=httpd:latest
```

**Explanation:**
This imperative approach directly creates a Deployment using command-line arguments. While faster, it is not ideal for production because it does not promote version control or GitOps practices.

To export YAML from this method:

```bash
kubectl create deployment httpd --image=httpd:latest --dry-run=client -o yaml > httpd-deployment.yaml
```

---

## Final Status

* Deployment Name: `httpd` → ✅ Created
* Image: `httpd:latest` → ✅ Correct
* Pods: Running → ✅ Healthy
* kubectl Connectivity: Verified → ✅

---

## Conclusion

The Apache HTTPD application was successfully deployed to the Kubernetes cluster using a Deployment resource. All validation checks confirm that the Deployment and its Pod are running as expected. Declarative YAML-based deployment was used following Kubernetes best practices.

---

## Interview Notes

* Always prefer Deployments over standalone Pods in production.
* Verify cluster access before applying workloads.
* Use declarative manifests for repeatable and auditable deployments.

