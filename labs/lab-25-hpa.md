# LAB 25 - Kubernetes Horizontal Pod Autoscaler (HPA)

## Objective

In this lab, you will learn:

- What is Horizontal Pod Autoscaler
- How HPA works
- Create HPA
- Generate CPU load
- Watch Pods scale automatically
- Understand minReplicas and maxReplicas

---

# What is Horizontal Pod Autoscaler?

Horizontal Pod Autoscaler automatically changes the number of Pods in a Deployment based on resource usage.

Example:

Normal traffic:

```
Users
 |
 |
2 Pods
```

High traffic:

```
Users
 |
 |
5 Pods
```

Kubernetes automatically creates more Pods.

---

# Why Do We Need HPA?

Imagine an online shopping website.

Normal day:

```
100 users

2 Pods are enough
```

Festival sale:

```
10,000 users

2 Pods cannot handle traffic
```

HPA:

```
Creates more Pods automatically
```

When traffic decreases:

```
Removes extra Pods
```

---

# HPA Flow

```
User Traffic

     |

Application Pods

     |

CPU Usage Increases

     |

Metrics Server

     |

HPA Controller

     |

Create More Pods

```

---

# HPA Components

HPA requires:

## 1. Deployment

HPA scales Deployments.

Example:

```
nginx Deployment
```

---

## 2. Metrics Server

Provides:

```
CPU usage

Memory usage
```

---

## 3. Resource Requests

HPA calculates usage based on requests.

Example:

```yaml
resources:
  requests:
    cpu: 100m
```

---

# HPA Configuration

Example:

```yaml
minReplicas: 2

maxReplicas: 5

targetCPUUtilizationPercentage: 50
```

Meaning:

Minimum:

```
2 Pods
```

Maximum:

```
5 Pods
```

When CPU reaches:

```
50%
```

Scale up.

---

# Lab Steps

# Step 1 - Verify Metrics Server

Run:

```bash
kubectl top nodes
```

Expected:

```
CPU
MEMORY
```

---

# Step 2 - Create Deployment

Create:

```bash
touch yaml/lab-25/hpa-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: hpa-demo


spec:

  replicas: 2


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

        image: nginx


        resources:

          requests:

            cpu: 100m
```

---

# Step 3 - Deploy Application

Run:

```bash
kubectl apply -f yaml/lab-25/hpa-deployment.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
2 nginx Pods Running
```

---

# Step 4 - Create HPA

Create:

```bash
touch yaml/lab-25/hpa.yaml
```

Add:

```yaml
apiVersion: autoscaling/v2

kind: HorizontalPodAutoscaler


metadata:

  name: nginx-hpa


spec:

  scaleTargetRef:

    apiVersion: apps/v1

    kind: Deployment

    name: hpa-demo


  minReplicas: 2

  maxReplicas: 5


  metrics:

  - type: Resource

    resource:

      name: cpu

      target:

        type: Utilization

        averageUtilization: 50
```

---

# Step 5 - Apply HPA

Run:

```bash
kubectl apply -f yaml/lab-25/hpa.yaml
```

---

Check:

```bash
kubectl get hpa
```

Expected:

```
NAME        REFERENCE

nginx-hpa   Deployment/hpa-demo
```

---

# Step 6 - Generate CPU Load

Create a temporary Pod:

```bash
kubectl run load-generator \
--image=busybox \
-- /bin/sh -c "while true; do wget -q -O- http://hpa-demo; done"
```

---

# Step 7 - Watch HPA Scaling

Run:

```bash
kubectl get hpa -w
```

You will see:

```
TARGET CPU

50%

80%

100%
```

---

# Step 8 - Watch Pods Increase

Run:

```bash
kubectl get pods -w
```

Expected:

```
hpa-demo-xxxx

hpa-demo-yyyy

hpa-demo-zzzz
```

Replica count increases.

---

# Step 9 - Stop Load

Delete load generator:

```bash
kubectl delete pod load-generator
```

Wait.

HPA reduces replicas.

---

# Verification

Check Deployment:

```bash
kubectl get deployment
```

Check HPA:

```bash
kubectl get hpa
```

Check Pods:

```bash
kubectl get pods
```

Check metrics:

```bash
kubectl top pods
```

---

# Troubleshooting

## HPA Shows Unknown

Problem:

```
TARGETS unknown
```

Solution:

Check Metrics Server:

```bash
kubectl top pods
```

---

## Pods Not Scaling

Check:

```bash
kubectl describe hpa nginx-hpa
```

Look for:

```
Events
```

---

# Interview Questions

## 1. What is HPA?

HPA automatically adjusts Pod replicas based on resource usage.

---

## 2. What does HPA require?

- Deployment
- Metrics Server
- Resource requests

---

## 3. Difference between Vertical and Horizontal Scaling?

Horizontal:

Increase number of Pods.

Vertical:

Increase CPU or memory of a Pod.

---

## 4. What happens when traffic decreases?

HPA removes extra Pods until minimum replicas are reached.

---

## 5. Which API is used by HPA?

Autoscaling API.

Example:

```
autoscaling/v2
```

---

# Commands Learned

```bash
kubectl get hpa

kubectl describe hpa

kubectl top pods

kubectl get deployment

kubectl scale

kubectl apply -f
```

---

# Cleanup

Delete HPA:

```bash
kubectl delete hpa nginx-hpa
```

Delete Deployment:

```bash
kubectl delete deployment hpa-demo
```

Delete Load Generator:

```bash
kubectl delete pod load-generator
```

---

# Navigation

Previous:

[Lab 24 - Metrics Server](lab-24-metrics-server.md)

Next:

[Lab 26 - ServiceAccount](lab-26-serviceaccount.md)

---

# Lab Completed ✅

You learned:

- Horizontal Pod Autoscaler
- Automatic Pod scaling
- CPU-based scaling
- Production autoscaling concepts