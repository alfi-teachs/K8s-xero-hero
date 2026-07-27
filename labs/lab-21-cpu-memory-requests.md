# LAB 21 - Kubernetes CPU and Memory Requests

## Objective

In this lab, you will learn:

- What are CPU requests
- What are Memory requests
- How Kubernetes reserves resources
- How Scheduler uses resource requests
- Create Pods with resource requirements

---

# Why Do We Need Resource Requests?

Every Node has limited resources.

Example:

```
Worker Node

CPU: 4 cores

Memory: 8GB
```

Multiple applications run on the same Node.

Kubernetes needs to know:

- How much CPU does a Pod need?
- How much memory does a Pod need?

This is done using requests.

---

# What is a Resource Request?

A request is the minimum amount of resource guaranteed for a container.

Example:

```yaml
resources:

  requests:

    cpu: "500m"

    memory: "512Mi"
```

Meaning:

```
CPU:
500 millicores

Memory:
512 MB
```

---

# CPU Units

Kubernetes measures CPU as:

```
1 CPU = 1000 millicores
```

Examples:

```
100m = 0.1 CPU

500m = 0.5 CPU

1000m = 1 CPU
```

---

# Memory Units

Examples:

```
128Mi

512Mi

1Gi
```

Mi = Mebibytes

Gi = Gibibytes

---

# How Scheduler Uses Requests

Flow:

```
Pod Created

      |

Scheduler checks requests

      |

Find Node with enough resources

      |

Pod scheduled

```

---

# Example

Node:

```
CPU Available:
4 CPU


Memory:
8Gi
```

Pod Request:

```
CPU:
1 CPU

Memory:
2Gi
```

Scheduler checks:

```
4 CPU >= 1 CPU

8Gi >= 2Gi
```

Pod can run.

---

# Requests vs Limits

| Requests | Limits |
|-|-|
| Minimum guaranteed resource | Maximum allowed resource |
| Used during scheduling | Used during runtime |
| Scheduler decision | Container restriction |

---

# Prerequisites

Check Nodes:

```bash
kubectl get nodes
```

---

# Lab Steps

# Step 1 - Check Node Resources

Run:

```bash
kubectl describe node minikube
```

Look for:

```
Capacity

CPU

Memory
```

---

# Step 2 - Create Pod Without Requests

Create:

```bash
touch yaml/lab-21/no-request-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: no-request-demo


spec:

  containers:

  - name: nginx

    image: nginx
```

---

Create:

```bash
kubectl apply -f yaml/lab-21/no-request-pod.yaml
```

Check:

```bash
kubectl get pods
```

---

# Step 3 - Create Pod With Requests

Create:

```bash
touch yaml/lab-21/resource-request-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: resource-request-demo


spec:

  containers:

  - name: nginx

    image: nginx


    resources:

      requests:

        cpu: "500m"

        memory: "512Mi"
```

---

# Step 4 - Deploy Pod

Run:

```bash
kubectl apply -f yaml/lab-21/resource-request-pod.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
resource-request-demo   Running
```

---

# Step 5 - View Resource Requests

Run:

```bash
kubectl describe pod resource-request-demo
```

Look for:

```
Requests:

cpu: 500m

memory: 512Mi
```

---

# Step 6 - Check Node Allocation

Run:

```bash
kubectl describe node minikube
```

Look for:

```
Allocated resources
```

You will see the requested CPU and memory.

---

# Step 7 - Test High Resource Request

Create:

```bash
touch yaml/lab-21/high-request-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: high-resource-demo


spec:

  containers:

  - name: nginx

    image: nginx


    resources:

      requests:

        cpu: "10"

        memory: "20Gi"
```

---

Apply:

```bash
kubectl apply -f yaml/lab-21/high-request-pod.yaml
```

Check:

```bash
kubectl get pods
```

You may see:

```
Pending
```

Why?

Because Node does not have enough resources.

---

# Verification

Check Pods:

```bash
kubectl get pods
```

Check Pod details:

```bash
kubectl describe pod resource-request-demo
```

Check Node resources:

```bash
kubectl describe node minikube
```

---

# Troubleshooting

## Pod stuck in Pending

Check:

```bash
kubectl describe pod POD_NAME
```

Look for:

```
Insufficient cpu

Insufficient memory
```

---

## Check Node Capacity

Run:

```bash
kubectl describe node minikube
```

---

# Interview Questions

## 1. What are CPU and Memory requests?

Requests define the minimum resources guaranteed for a container.

---

## 2. Who uses resource requests?

The Kubernetes Scheduler uses requests to select Nodes.

---

## 3. Difference between Requests and Limits?

Requests:

Minimum guaranteed resources.

Limits:

Maximum resources allowed.

---

## 4. What happens if a Node does not have enough resources?

The Pod remains in Pending state.

---

## 5. What is 500m CPU?

500 millicores = 0.5 CPU.

---

# Commands Learned

```bash
kubectl describe node

kubectl describe pod

kubectl get pods

kubectl apply -f
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod no-request-demo
```

```bash
kubectl delete pod resource-request-demo
```

```bash
kubectl delete pod high-resource-demo
```

---

# Navigation

Previous:

[Lab 20 - Node Affinity](lab-20-node-affinity.md)

Next:

[Lab 22 - CPU and Memory Limits](lab-22-cpu-memory-limits.md)

---

# Lab Completed ✅

You learned:

- CPU requests
- Memory requests
- Scheduler resource decisions
- Resource planning