# LAB 24 - Kubernetes Metrics Server

## Objective

In this lab, you will learn:

- What Metrics Server is
- Why Kubernetes needs metrics
- Install Metrics Server
- Check Node CPU and Memory usage
- Check Pod CPU and Memory usage
- Prepare cluster for HPA

---

# What is Metrics Server?

Metrics Server collects resource usage information from Kubernetes Nodes and Pods.

It collects:

- CPU usage
- Memory usage

Example:

```
Pod

CPU: 200m

Memory: 300Mi
```

Kubernetes uses this information for:

- Monitoring
- Autoscaling
- Resource decisions

---

# Why Do We Need Metrics Server?

Without Metrics Server:

```
kubectl top pods

ERROR
Metrics API not available
```

With Metrics Server:

```
kubectl top pods

NAME       CPU    MEMORY

nginx      5m     20Mi
```

---

# Kubernetes Metrics Flow

```
Container

    |

Kubelet

    |

Metrics Server

    |

Kubernetes API

    |

kubectl top

```

---

# Metrics Server Use Cases

## 1. Monitoring

View:

- Pod CPU
- Pod Memory
- Node CPU
- Node Memory

---

## 2. Horizontal Pod Autoscaler

HPA uses metrics:

Example:

```
CPU > 70%

Create more Pods
```

---

# Prerequisites

Check cluster:

```bash
kubectl get nodes
```

Check Minikube:

```bash
minikube status
```

---

# Lab Steps

# Step 1 - Check Metrics Server Status

Run:

```bash
kubectl get deployment metrics-server -n kube-system
```

If installed:

```
NAME             READY

metrics-server   1/1
```

---

# Step 2 - Enable Metrics Server in Minikube

Run:

```bash
minikube addons enable metrics-server
```

Expected:

```
The 'metrics-server' addon is enabled
```

---

# Step 3 - Verify Metrics Server Pod

Run:

```bash
kubectl get pods -n kube-system | grep metrics
```

Expected:

```
metrics-server-xxxxx   Running
```

---

# Step 4 - Wait Until Ready

Run:

```bash
kubectl get pods -n kube-system -w
```

Wait until:

```
Running
```

Press:

```
CTRL + C
```

---

# Step 5 - Check Node Metrics

Run:

```bash
kubectl top nodes
```

Example:

```
NAME       CPU     MEMORY

minikube   200m    1500Mi
```

---

# Step 6 - Create Test Deployment

Create:

```bash
touch yaml/lab-24/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: nginx-metrics-demo


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

            memory: 128Mi
```

---

# Step 7 - Deploy Application

Run:

```bash
kubectl apply -f yaml/lab-24/nginx-deployment.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
nginx-metrics-demo-xxxxx

Running
```

---

# Step 8 - View Pod Metrics

Run:

```bash
kubectl top pods
```

Example:

```
NAME                     CPU     MEMORY

nginx-metrics-demo       1m      10Mi
```

---

# Step 9 - View Specific Pod Metrics

Run:

```bash
kubectl top pod POD_NAME
```

Example:

```bash
kubectl top pod nginx-metrics-demo-xxxxx
```

---

# Step 10 - View Metrics API

Run:

```bash
kubectl get --raw /apis/metrics.k8s.io/
```

Expected:

```
{
 "kind":"APIGroup"
}
```

---

# Verification

Check Metrics Server:

```bash
kubectl get pods -n kube-system | grep metrics
```

Check Nodes:

```bash
kubectl top nodes
```

Check Pods:

```bash
kubectl top pods
```

---

# Troubleshooting

## Error: Metrics API not available

Check:

```bash
kubectl get pods -n kube-system
```

---

Restart Metrics Server:

```bash
kubectl rollout restart deployment metrics-server -n kube-system
```

---

## Metrics Server Pod Not Running

Check logs:

```bash
kubectl logs -n kube-system deployment/metrics-server
```

---

# Interview Questions

## 1. What is Metrics Server?

Metrics Server collects CPU and Memory usage from Kubernetes resources.

---

## 2. Is Metrics Server a monitoring tool?

No.

Metrics Server provides short-term resource metrics.

Tools like Prometheus are used for detailed monitoring.

---

## 3. Why is Metrics Server required for HPA?

HPA needs CPU and Memory metrics to decide when to scale Pods.

---

## 4. Difference between Metrics Server and Prometheus?

Metrics Server:

- Basic resource metrics
- Used by Kubernetes autoscaling

Prometheus:

- Advanced monitoring
- Stores historical metrics

---

# Commands Learned

```bash
kubectl top nodes

kubectl top pods

kubectl get pods -n kube-system

kubectl logs

kubectl rollout restart
```

---

# Cleanup

Delete Deployment:

```bash
kubectl delete deployment nginx-metrics-demo
```

Disable addon:

```bash
minikube addons disable metrics-server
```

---

# Navigation

Previous:

[Lab 23 - Namespace](lab-23-namespace.md)

Next:

[Lab 25 - Horizontal Pod Autoscaler](lab-25-hpa.md)

---

# Lab Completed ✅

You learned:

- Metrics Server
- CPU and Memory metrics
- kubectl top command
- HPA preparation