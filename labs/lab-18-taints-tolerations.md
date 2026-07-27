# LAB 18 - Kubernetes Taints and Tolerations

## Objective

In this lab, you will learn:

- What are Taints
- What are Tolerations
- How Kubernetes restricts Pod scheduling
- How to allow specific Pods on restricted Nodes
- How scheduling decisions happen

---

# What is Scheduling?

Scheduling means deciding:

"On which Node should this Pod run?"

Example:

```
        Kubernetes Cluster


        Control Plane

              |

        Scheduler

              |

   -----------------------

   Node 1       Node 2

     |            |

   Pod A        Pod B

```

The Scheduler chooses the best Node for a Pod.

---

# What is a Taint?

A Taint is a rule applied to a Node that prevents Pods from running on it.

Think:

```
Node

"Only special Pods are allowed here"
```

---

# Real World Example

Imagine:

```
Node 1

GPU Machine

Taint:

Only AI workloads allowed
```

Normal applications should not run there.

---

# Taint Syntax

Command:

```bash
kubectl taint nodes NODE_NAME key=value:effect
```

Example:

```bash
kubectl taint nodes worker1 app=gpu:NoSchedule
```

Meaning:

```
Node worker1

Reject Pods

unless they tolerate this taint
```

---

# Taint Effects

## NoSchedule

New Pods cannot schedule.

Existing Pods continue running.

---

## PreferNoSchedule

Kubernetes tries to avoid placing Pods.

---

## NoExecute

Existing Pods without toleration are removed.

---

# What is Toleration?

A Toleration allows a Pod to run on a Node with a matching Taint.

Example:

Node:

```
Taint:

app=gpu:NoSchedule
```

Pod:

```
Toleration:

app=gpu
```

Result:

```
Pod can run on Node
```

---

# Taint and Toleration Relationship

```
Node

   |

 Taint

   |

Blocks Pods


       


Pod

   |

Toleration

   |

Allows scheduling

```

---

# Prerequisites

Check Nodes:

```bash
kubectl get nodes
```

---

# Lab Steps

# Step 1 - Check Nodes

Run:

```bash
kubectl get nodes
```

Example:

```
NAME       STATUS

minikube   Ready
```

---

# Step 2 - Add Taint to Node

Find Node:

```bash
kubectl get nodes
```

Apply taint:

```bash
kubectl taint nodes minikube app=database:NoSchedule
```

Expected:

```
node/minikube tainted
```

---

# Step 3 - Create Normal Pod

Create:

```bash
touch yaml/lab-18/normal-pod.yaml
```

Open:

```bash
code yaml/lab-18/normal-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: normal-pod


spec:

  containers:

  - name: nginx

    image: nginx
```

---

# Step 4 - Deploy Pod

Run:

```bash
kubectl apply -f yaml/lab-18/normal-pod.yaml
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

Because Node has a taint.

---

# Step 5 - Check Reason

Run:

```bash
kubectl describe pod normal-pod
```

Look at Events:

Example:

```
had untolerated taint
```

---

# Step 6 - Create Pod With Toleration

Create:

```bash
touch yaml/lab-18/toleration-pod.yaml
```

Open:

```bash
code yaml/lab-18/toleration-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: toleration-pod


spec:

  tolerations:

  - key: "app"

    operator: "Equal"

    value: "database"

    effect: "NoSchedule"



  containers:

  - name: nginx

    image: nginx
```

---

# Step 7 - Deploy Toleration Pod

Run:

```bash
kubectl apply -f yaml/lab-18/toleration-pod.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
toleration-pod   Running
```

---

# Step 8 - Remove Taint

Cleanup:

```bash
kubectl taint nodes minikube app=database:NoSchedule-
```

The minus sign removes the taint.

---

# Verification

Check Pods:

```bash
kubectl get pods
```

Check Node:

```bash
kubectl describe node minikube
```

Look for:

```
Taints
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
untolerated taint
```

---

## Remove all taints

Command:

```bash
kubectl taint nodes minikube --all
```

---

# Interview Questions

## 1. What is a Taint?

A taint prevents Pods from being scheduled on a Node.

---

## 2. What is a Toleration?

A toleration allows a Pod to run on a tainted Node.

---

## 3. Difference between Node Selector and Taints?

Node Selector:

```
Choose where Pod should run
```

Taints:

```
Prevent Pods from running
```

---

## 4. Where are Taints used?

Examples:

- GPU Nodes
- Dedicated database Nodes
- Production workloads

---

# Commands Learned

```bash
kubectl taint nodes

kubectl describe node

kubectl describe pod

kubectl get pods

kubectl apply -f
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod normal-pod
kubectl delete pod toleration-pod
```

Remove taint:

```bash
kubectl taint nodes minikube app=database:NoSchedule-
```

---

# Navigation

Previous:

[Lab 17 - HostPath Volume](lab-17-hostpath.md)

Next:

[Lab 19 - Node Selector](lab-19-node-selector.md)

---

# Lab Completed ✅

You learned:

- Node Taints
- Pod Tolerations
- Scheduling restrictions
- Kubernetes scheduler behavior