🚀 Deployed a Guestbook Application on Kubernetes (With a Real Failure & Fix)

Recently worked on deploying a Guestbook application on Kubernetes using a three-tier architecture and intentionally walked through a real failure scenario to validate troubleshooting depth.

Sharing this as a practical Kubernetes reference, not a textbook example.

🏗️ Architecture Overview

Three-tier design:

Frontend – PHP-based web UI

Redis Master – Write backend

Redis Slaves – Read replicas

All components deployed as Kubernetes Deployments and exposed via Services.

📦 Redis Master (Backend – Write)
Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
        - name: master-redis-natural
          image: redis
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 100m
              memory: 100Mi

Service
apiVersion: v1
kind: Service
metadata:
  name: redis-master
spec:
  selector:
    app: redis
    role: master
  ports:
    - port: 6379
      targetPort: 6379

📦 Redis Slaves (Backend – Read)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
        - name: slave-redis-natural
          image: gcr.io/google_samples/gb-redisslave:v3
          ports:
            - containerPort: 6379
          env:
            - name: GET_HOSTS_FROM
              value: dns
          resources:
            requests:
              cpu: 100m
              memory: 100Mi

🌐 Frontend (UI Layer)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
        - name: php-redis-natural
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          ports:
            - containerPort: 80
          env:
            - name: GET_HOSTS_FROM
              value: dns
          resources:
            requests:
              cpu: 100m
              memory: 100Mi

NodePort Service
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: guestbook
    tier: frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30009

🚀 Deployment Commands
kubectl apply -f Redis-Master-Deployment.yml
kubectl apply -f Redis-Master-Service.yml
kubectl apply -f Redis-Slave-Deployment.yml
kubectl apply -f Redis-Slave-Service.yml
kubectl apply -f Frontend-Deployment.yml
kubectl apply -f Frontend-Service-NodePort.yml

🚨 Real Issue Encountered
kubectl get pods

redis-master   ImagePullBackOff

kubectl describe pod redis-master-xxxx

Failed to pull image "docker.io/library/redis:latest"
unexpected EOF

🔎 Root Cause

Cluster had restricted / unstable access to Docker Hub

Redis master image (redis:latest) could not be pulled

Redis slave & frontend worked (GCR images)

✅ Fix Applied (Production-Correct)
kubectl set image deployment redis-master \
  master-redis-natural=k8s.gcr.io/redis:e2e

kubectl rollout status deployment redis-master

deployment "redis-master" successfully rolled out

🧠 Key Learnings

ImagePullBackOff is a registry access issue, not an app issue

Containers that never start → no logs

Frontend can load even if backend is down (stateless design)

Fix Deployments, not Pods

Rolling updates temporarily create multiple Pods

Enterprise clusters often block Docker Hub

☁️ Cloud / AWS Mapping

Kubernetes Deployments → EKS

Redis → ElastiCache

Docker Hub → ECR

NodePort → ALB / NLB

Real-world incident pattern:
App outages caused by Docker Hub throttling → fixed by migrating images to private ECR.

🔚 Final Thought

This wasn’t about deploying a guestbook —
it was about understanding Kubernetes behavior under failure and fixing it the right way.

#Kubernetes #DevOps #CloudEngineering #EKS #Redis #Containers #SRE #PlatformEngineering #HandsOn
