# LAB 20 - Kubernetes Node Affinity

## Objective

In this lab, you will learn:

- What is Node Affinity
- Difference between Node Selector and Node Affinity
- Required Node Affinity
- Preferred Node Affinity
- Schedule Pods using advanced rules

---

# What is Node Affinity?

Node Affinity controls where Pods are scheduled based on Node labels.

It is an advanced version of Node Selector.

Example:

```
Node 1

disk=ssd


Node 2

disk=hdd
```

Application:

```
Needs SSD storage
```

Node Affinity places the Pod on:

```
Node 1
```

---

# Node Selector vs Node Affinity

| Node Selector | Node Affinity |
|---|---|
| Simple matching | Advanced rules |
| Equal condition only | Multiple operators |
| Less flexible | More flexible |
| Basic scheduling | Production scheduling |

---

# Types of Node Affinity

Kubernetes provides two types:

---

# 1. RequiredDuringSchedulingIgnoredDuringExecution

Meaning:

The rule must be satisfied.

If no Node matches:

```
Pod stays Pending
```

Example:

```
Only run on SSD nodes
```

---

# 2. PreferredDuringSchedulingIgnoredDuringExecution

Meaning:

Try to satisfy the rule.

If not possible:

```
Pod can run elsewhere
```

Example:

```
Prefer SSD nodes
```

---

# Node Affinity Flow

```
              Pod

               |

        Node Affinity Rule

               |

          Kubernetes Scheduler

               |

        Matching Node Found

               |

              Pod Runs

```

---

# Prerequisites

Check Nodes:

```bash
kubectl get nodes
```

---

# Lab Steps

# Step 1 - Add Label to Node

Add storage label:

```bash
kubectl label nodes minikube disk=ssd
```

---

# Step 2 - Verify Label

Run:

```bash
kubectl get nodes --show-labels
```

Look for:

```
disk=ssd
```

---

# Step 3 - Create Required Node Affinity Pod

Create:

```bash
touch yaml/lab-20/required-affinity.yaml
```

Open:

```bash
code yaml/lab-20/required-affinity.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: required-affinity-demo


spec:

  affinity:

    nodeAffinity:

      requiredDuringSchedulingIgnoredDuringExecution:

        nodeSelectorTerms:

        - matchExpressions:

          - key: disk

            operator: In

            values:

            - ssd



  containers:

  - name: nginx

    image: nginx
```

---

# Step 4 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-20/required-affinity.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
required-affinity-demo   Running
```

---

# Step 5 - Verify Node Placement

Run:

```bash
kubectl get pod required-affinity-demo -o wide
```

Example:

```
NAME                     NODE
required-affinity-demo   minikube
```

---

# Step 6 - Test Affinity Failure

Remove label:

```bash
kubectl label nodes minikube disk-
```

Delete Pod:

```bash
kubectl delete pod required-affinity-demo
```

Create again:

```bash
kubectl apply -f yaml/lab-20/required-affinity.yaml
```

Check:

```bash
kubectl get pods
```

Output:

```
Pending
```

Because:

```
No Node matches disk=ssd
```

---

# Step 7 - Add Label Back

Run:

```bash
kubectl label nodes minikube disk=ssd
```

Delete Pod:

```bash
kubectl delete pod required-affinity-demo
```

Create:

```bash
kubectl apply -f yaml/lab-20/required-affinity.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

```
Running
```

---

# Step 8 - Preferred Node Affinity

Create:

```bash
touch yaml/lab-20/preferred-affinity.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: preferred-affinity-demo


spec:

  affinity:

    nodeAffinity:

      preferredDuringSchedulingIgnoredDuringExecution:

      - weight: 1

        preference:

          matchExpressions:

          - key: disk

            operator: In

            values:

            - ssd



  containers:

  - name: nginx

    image: nginx
```

---

Create:

```bash
kubectl apply -f yaml/lab-20/preferred-affinity.yaml
```

Check:

```bash
kubectl get pods -o wide
```

---

# Verification

Run:

```bash
kubectl get pods
```

```bash
kubectl get nodes --show-labels
```

```bash
kubectl describe pod required-affinity-demo
```

---

# Troubleshooting

## Pod stuck in Pending

Check:

```bash
kubectl describe pod required-affinity-demo
```

Look for:

```
didn't match node affinity
```

---

## Check Node Labels

Run:

```bash
kubectl get nodes --show-labels
```

---

# Interview Questions

## 1. What is Node Affinity?

Node Affinity is a scheduling feature that controls Pod placement using Node labels.

---

## 2. Difference between Required and Preferred?

Required:

Pod must match.

Preferred:

Scheduler tries but can ignore.

---

## 3. Difference between Node Selector and Node Affinity?

Node Selector:

Simple key-value matching.

Node Affinity:

Advanced scheduling rules.

---

## 4. Where is Node Affinity used?

Examples:

- Database workloads
- GPU workloads
- High performance applications

---

# Commands Learned

```bash
kubectl label nodes

kubectl get nodes --show-labels

kubectl get pods -o wide

kubectl describe pod

kubectl apply -f
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod required-affinity-demo
```

```bash
kubectl delete pod preferred-affinity-demo
```

Remove label:

```bash
kubectl label nodes minikube disk-
```

---

# Navigation

Previous:

[Lab 19 - Node Selector](lab-19-node-selector.md)

Next:

[Lab 21 - CPU and Memory Requests](lab-21-cpu-memory-requests.md)

---

# Lab Completed ✅

You learned:

- Node Affinity
- Required scheduling rules
- Preferred scheduling rules
- Advanced Pod placement