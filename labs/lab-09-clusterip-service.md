# LAB 09 - Kubernetes ClusterIP Service

## Objective

In this lab, you will learn:

- What is a Kubernetes Service
- Why Services are required
- What is ClusterIP
- Create a Deployment
- Create a ClusterIP Service
- Access application internally
- Understand Service discovery

---

# What is a Kubernetes Service?

A Service provides stable network access to Pods.

Pods are temporary.

Example:

Pod:

```
nginx-pod-abc123
```

If the Pod crashes:

```
nginx-pod-xyz789
```

New Pod gets a new IP address.

Service solves this problem.

---

# Why Do We Need Services?

Without Service:

```
User

 |
 |
Pod IP

10.244.1.5
```

Problem:

Pod IP changes when Pod restarts.

---

With Service:

```
User

 |

Service IP

 |

Pod
Pod
Pod
```

Service provides:

- Stable IP
- Load balancing
- Service discovery

---

# Types of Kubernetes Services

Kubernetes has:

## ClusterIP

Internal communication only.

Example:

```
Backend ---> Database
```

---

## NodePort

Access from outside the cluster.

Example:

```
Browser ---> Node IP
```

---

## LoadBalancer

External cloud load balancer.

Example:

```
Internet ---> AWS ELB ---> Service
```

---

# ClusterIP Architecture

```
             ClusterIP Service

                    |

          +---------+---------+

          |         |         |

        Pod       Pod       Pod
```

---

# Prerequisites

Check cluster:

```bash
kubectl get nodes
```

Expected:

```
minikube Ready
```

---

# Lab Steps

## Step 1 - Create Deployment

Create file:

```bash
touch yaml/lab-09/nginx-deployment.yaml
```

Open:

```bash
code yaml/lab-09/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-service-demo


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

      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80
```

---

## Step 2 - Create Deployment

Run:

```bash
kubectl apply -f yaml/lab-09/nginx-deployment.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
3 nginx Pods Running
```

---

# Step 3 - Create ClusterIP Service

Create:

```bash
touch yaml/lab-09/nginx-service.yaml
```

Open:

```bash
code yaml/lab-09/nginx-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-clusterip-service


spec:

  type: ClusterIP


  selector:
    app: nginx


  ports:

  - port: 80
    targetPort: 80
```

---

# Understanding Service YAML

## Type

```yaml
type: ClusterIP
```

Creates internal-only Service.

---

## Selector

```yaml
selector:
  app: nginx
```

Service sends traffic to Pods having:

```
app=nginx
```

---

## Port

Service port:

```yaml
port: 80
```

Container port:

```yaml
targetPort: 80
```

Traffic flow:

```
Service:80

      |

Pod:80
```

---

# Step 4 - Create Service

Run:

```bash
kubectl apply -f yaml/lab-09/nginx-service.yaml
```

Expected:

```
service/nginx-clusterip-service created
```

---

# Step 5 - Check Service

Run:

```bash
kubectl get service
```

Expected:

```
NAME                       TYPE

nginx-clusterip-service    ClusterIP
```

---

# Step 6 - Get Service IP

Run:

```bash
kubectl get svc nginx-clusterip-service
```

Example:

```
CLUSTER-IP

10.96.120.50
```

This IP works only inside Kubernetes.

---

# Step 7 - Test Service Internally

Create a temporary test Pod:

```bash
kubectl run curl-test \
--image=curlimages/curl \
-it \
--rm \
-- sh
```

Inside the Pod:

Test Service:

```bash
curl nginx-clusterip-service
```

Expected:

```
Welcome to nginx!
```

Exit:

```bash
exit
```

---

# Step 8 - Check Service Endpoints

Run:

```bash
kubectl get endpoints
```

Expected:

```
nginx-clusterip-service

10.x.x.x
10.x.x.x
10.x.x.x
```

These are Pod IP addresses.

---

# Verification

Run:

```bash
kubectl get deployment

kubectl get pods

kubectl get service

kubectl get endpoints
```

Everything should be running.

---

# Troubleshooting

## Service not connecting

Check labels:

```bash
kubectl get pods --show-labels
```

Service selector:

```yaml
selector:
  app: nginx
```

Pod label must match:

```yaml
labels:
  app: nginx
```

---

## No endpoints

Check:

```bash
kubectl describe service nginx-clusterip-service
```

---

# Interview Questions

## 1. What is Kubernetes Service?

A Service provides stable networking to Pods.

---

## 2. Why do we need Services?

Because Pod IP addresses are temporary.

---

## 3. What is ClusterIP?

Default Service type used for internal communication.

---

## 4. How does Service find Pods?

Using labels and selectors.

---

## 5. Can ClusterIP be accessed from internet?

No.

It is internal to the cluster.

---

# Commands Learned

```bash
kubectl apply -f file.yaml

kubectl get service

kubectl get svc

kubectl get endpoints

kubectl describe service service-name

kubectl run curl-test
```

---

# Cleanup

Delete Service:

```bash
kubectl delete service nginx-clusterip-service
```

Delete Deployment:

```bash
kubectl delete deployment nginx-service-demo
```

---

# Navigation

Previous:

[Lab 08 - Rollback](lab-08-rollback.md)

Next:

[Lab 10 - NodePort Service](lab-10-nodeport-service.md)

---

# Lab Completed ✅

You learned:

- Kubernetes Service
- ClusterIP
- Service discovery
- Labels and selectors
- Internal communication