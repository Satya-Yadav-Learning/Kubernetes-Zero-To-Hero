Below is a direct mapping of your kubectl/Kubernetes Q&A to real AWS incident scenarios, written exactly the way on-call engineers, SREs, and AWS interviewers expect answers.

This is not theory — these are real incidents that happen in AWS EKS environments.

Mapping: kubectl & Kubernetes → Real AWS Incident Scenarios

Platform: AWS EKS + EC2 + VPC + IAM
Perspective: On-call / Production / Interview

Incident 1: kubectl Works, But Cluster Is Unreachable (SEV-1)
Scenario

kubectl version --client works

kubectl get pods hangs or times out

AWS Root Cause

EKS API server endpoint not reachable

VPN / Bastion / Jump host network issue

Private endpoint without proper routing

How You Diagnose
kubectl cluster-info
kubectl config current-context

AWS-Level Checks

VPC route tables

Security Groups (API server access)

EKS endpoint access (public/private)

Interview Mapping

“kubectl is only a client. If the EKS control plane is unreachable, kubectl cannot operate.”

Incident 2: Repository Not Found / Access Denied to Cluster (SEV-2)
Scenario

kubectl installed and configured

Error: Forbidden or You must be logged in to the server

AWS Root Cause

IAM user/role not mapped in aws-auth ConfigMap

Missing RBAC permissions

How You Diagnose
kubectl auth can-i get pods
kubectl config get-contexts

AWS Fix

Update aws-auth ConfigMap

Attach correct IAM role

Use IRSA for service access

Interview Mapping

“In EKS, IAM controls authentication, RBAC controls authorization.”

Incident 3: Engineer Ran Command on Wrong Cluster (SEV-0 Near Miss)
Scenario

Intended to deploy to dev

Accidentally modified prod

AWS Root Cause

Multiple kubeconfig contexts

No context awareness

How You Prevent
kubectl config get-contexts
kubectl config use-context dev

AWS Best Practice

Separate AWS accounts per environment

Separate EKS clusters

Read-only prod access

Interview Mapping

“Context isolation is critical to prevent production outages.”

Incident 4: kubectl Installed, Pods Not Scheduling (SEV-1)
Scenario

kubectl apply works

Pods stuck in Pending

AWS Root Cause

EC2 node group out of capacity

AZ capacity shortage

Wrong instance types

How You Diagnose
kubectl describe pod <pod>
kubectl get nodes

AWS Fix

Scale node group

Use mixed instance policy

Enable Cluster Autoscaler

Incident 5: kubectl Works on Jump Host but Not on Laptop (SEV-3)
Scenario

On-call engineer cannot run kubectl locally

AWS Root Cause

kubeconfig missing locally

IAM credentials expired

Fix
aws eks update-kubeconfig --region us-east-1 --name cluster-name

Interview Mapping

“kubeconfig is machine-specific; kubectl installation alone is insufficient.”

Incident 6: kubectl Hangs During Production Outage (SEV-0)
Scenario

ALB returning 5xx

kubectl commands not responding

AWS Root Cause

EKS control plane degraded

Network partition

etcd pressure

Recovery Path

Use GitOps rollback

AWS Console (last resort)

CloudWatch + ALB metrics

Interview Mapping

“kubectl is not a recovery system; it depends on the control plane.”

Incident 7: Sudden Pod Failures After Deployment (SEV-1)
Scenario

Pods crash after deployment

kubectl shows restarts

AWS Root Cause

latest image pulled

Breaking change introduced

How You Diagnose
kubectl describe pod
kubectl logs <pod>

AWS Fix

Pin image versions

Roll back via Deployment

Use canary / blue-green

Incident 8: High AWS Bill Overnight (Cost Incident)
Scenario

Cost spike without traffic increase

Root Cause

No resource limits

HPA scaling aggressively

Cluster Autoscaler runaway

kubectl Check
kubectl describe hpa
kubectl top pods

AWS Fix

Resource quotas

HPA max limits

Spot instances

Incident 9: Security Breach – Unauthorized kubectl Usage (SEV-0)
Scenario

Unknown resources created

AWS Root Cause

Over-permissive IAM role

Shared kubeconfig

Fix

Rotate credentials

Enforce IAM + RBAC

Enable CloudTrail + audit logs

Interview Mapping

“kubectl access equals cluster control.”

Incident 10: Production Change Without Audit Trail
Scenario

Manual kubectl apply

No Git history

Root Cause

No GitOps

Engineers modifying prod directly

AWS / Kubernetes Fix

ArgoCD / Flux

Read-only kubectl in prod

High-Value AWS Mapping Table
Kubernetes / kubectl	AWS Component
kubectl	Client tool
API Server	EKS Control Plane
Node	EC2
Auth	IAM
AuthZ	RBAC
Service	ALB / NLB
Logs	CloudWatch
GitOps	CI/CD + ArgoCD
Interview Power Answer (Use This)

“Most AWS EKS incidents are not kubectl issues — they are IAM, networking, or capacity issues. kubectl is only the visibility and control interface.”
