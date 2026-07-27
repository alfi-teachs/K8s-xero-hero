# LAB 22 - Kubernetes CPU and Memory Limits

## Objective

In this lab, you will learn:

- What are CPU limits
- What are Memory limits
- Difference between Requests and Limits
- How Kubernetes controls containers
- Understand OOMKilled errors

---

# Why Do We Need Limits?

Imagine:

```
Node

CPU: 4 cores
Memory: 8GB
```

Application A uses:

```
7GB Memory
```

Other applications may crash.

Kubernetes prevents this using limits.

---

# What is a Resource Limit?

A limit defines the maximum resource a container can consume.

Example:

```yaml
resources:

  limits:

    cpu: "1"

    memory: "1Gi"
```

Meaning:

```
Maximum CPU:
1 CPU


Maximum Memory:
1GB
```

---

# Requests vs Limits

| Requests | Limits |
|---|---|
| Minimum guaranteed resources | Maximum allowed resources |
| Used by Scheduler | Enforced by Kubelet |
| Pod placement decision | Container restriction |

Example:

```yaml
resources:

  requests:

    cpu: "500m"

    memory: "512Mi"


  limits:

    cpu: "1"

    memory: "1Gi"
```

Meaning:

Container gets:

Minimum:

```
CPU: 0.5

Memory: 512MB
```

Maximum:

```
CPU: 1

Memory: 1GB
```

---

# CPU Limit Behavior

If container tries to use more CPU:

Example:

Limit:

```
CPU: 1
```

Container uses:

```
2 CPU
```

Kubernetes:

```
Throttles CPU usage
```

Container continues running.

---

# Memory Limit Behavior

Memory is different.

If container exceeds memory limit:

Example:

Limit:

```
512Mi
```

Container uses:

```
700Mi
```

Kubernetes kills container.

Status:

```
OOMKilled
```

OOM means:

```
Out Of Memory
```

---

# Lab Steps

# Step 1 - Create Pod Without Limits

Create:

```bash
touch yaml/lab-22/no-limit-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: no-limit-demo


spec:

  containers:

  - name: nginx

    image: nginx
```

---

Create:

```bash
kubectl apply -f yaml/lab-22/no-limit-pod.yaml
```

Check:

```bash
kubectl get pods
```

---

# Step 2 - Create Pod With Resource Limits

Create:

```bash
touch yaml/lab-22/resource-limit-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: resource-limit-demo


spec:

  containers:

  - name: nginx

    image: nginx


    resources:

      requests:

        cpu: "500m"

        memory: "512Mi"


      limits:

        cpu: "1"

        memory: "1Gi"
```

---

# Step 3 - Deploy Pod

Run:

```bash
kubectl apply -f yaml/lab-22/resource-limit-pod.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
resource-limit-demo   Running
```

---

# Step 4 - View Resource Configuration

Run:

```bash
kubectl describe pod resource-limit-demo
```

Look for:

```
Requests:

cpu: 500m
memory: 512Mi


Limits:

cpu: 1
memory: 1Gi
```

---

# Step 5 - Check Node Allocation

Run:

```bash
kubectl describe node minikube
```

Look for:

```
Allocated resources
```

---

# Step 6 - Generate Memory Usage Test

Create:

```bash
touch yaml/lab-22/memory-test.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: memory-test


spec:

  containers:

  - name: memory-container

    image: polinux/stress


    args:

    - "--vm"

    - "1"

    - "--vm-bytes"

    - "600M"


    resources:

      limits:

        memory: "256Mi"
```

---

Create:

```bash
kubectl apply -f yaml/lab-22/memory-test.yaml
```

Check:

```bash
kubectl get pods
```

You may see:

```
CrashLoopBackOff
```

Check:

```bash
kubectl describe pod memory-test
```

You will see:

```
OOMKilled
```

---

# Verification

Check Pods:

```bash
kubectl get pods
```

Check details:

```bash
kubectl describe pod resource-limit-demo
```

Check node:

```bash
kubectl describe node minikube
```

---

# Troubleshooting

## Container Restarting

Check:

```bash
kubectl describe pod POD_NAME
```

Look for:

```
OOMKilled
```

---

## Check Container Logs

Run:

```bash
kubectl logs POD_NAME
```

---

# Interview Questions

## 1. What are Resource Limits?

Limits define the maximum CPU and Memory a container can consume.

---

## 2. What happens when CPU limit is exceeded?

CPU is throttled.

---

## 3. What happens when Memory limit is exceeded?

Container is killed with OOMKilled status.

---

## 4. Difference between Requests and Limits?

Requests:

Guaranteed resources.

Limits:

Maximum resources.

---

## 5. Why are limits important?

They prevent one application from consuming all Node resources.

---

# Commands Learned

```bash
kubectl describe pod

kubectl describe node

kubectl get pods

kubectl logs

kubectl apply -f
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod no-limit-demo
```

```bash
kubectl delete pod resource-limit-demo
```

```bash
kubectl delete pod memory-test
```

---

# Navigation

Previous:

[Lab 21 - CPU and Memory Requests](lab-21-cpu-memory-requests.md)

Next:

[Lab 23 - Namespace](lab-23-namespace.md)

---

# Lab Completed ✅

You learned:

- CPU limits
- Memory limits
- Resource protection
- OOMKilled troubleshooting