# LAB 33 - Pending Pods Troubleshooting

## Objective

In this lab, you will learn:

- What Pending status means
- Why Pods become Pending
- How Kubernetes scheduler works
- How to identify scheduling problems
- How to fix Pending Pods

---

# What is Pending State?

A Pod is in Pending state when:

```
Pod created

      |

Scheduler tries to find a node

      |

No suitable node found

      |

Pod remains Pending
```

The Pod object exists, but the container has not started.

---

# Pod Lifecycle

```
Pending

   |

Scheduled

   |

Container Creating

   |

Running

```

---

# Common Causes of Pending Pods

## 1. Not Enough CPU or Memory

Example:

Node has:

```
CPU Available: 1 Core
```

Pod requests:

```
CPU Required: 4 Cores
```

Result:

```
Pending
```

---

## 2. Node Selector Problem

Example:

Pod requires:

```yaml
nodeSelector:
  disk: ssd
```

But no node has:

```
disk=ssd
```

Result:

```
Pending
```

---

## 3. Taints and Tolerations

Node has:

```
NoSchedule taint
```

Pod does not tolerate it.

Result:

```
Pending
```

---

## 4. Persistent Volume Problem

Pod requires:

```
PVC
```

but no matching:

```
PV
```

exists.

---

## 5. Node Not Available

No healthy worker nodes available.

---

# Troubleshooting Flow

```
Pod Pending

    |

kubectl get pods

    |

kubectl describe pod

    |

Check Events

    |

Find Scheduling Issue

    |

Fix Configuration

```

---

# Lab Steps

# Step 1 - Create Pod With Impossible Resource Request

Create:

```bash
touch yaml/lab-33/pending-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: pending-pod


spec:

  containers:

  - name: nginx

    image: nginx

    resources:

      requests:

        memory: "100Gi"

        cpu: "50"
```

---

# Step 2 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-33/pending-pod.yaml
```

---

# Step 3 - Check Pod Status

Run:

```bash
kubectl get pods
```

Expected:

```
pending-pod

Pending
```

---

# Step 4 - Describe Pod

Run:

```bash
kubectl describe pod pending-pod
```

Look at:

```
Events
```

Example:

```
0/1 nodes available:
Insufficient cpu
```

or

```
Insufficient memory
```

---

# Step 5 - Check Node Resources

Run:

```bash
kubectl describe nodes
```

Look for:

```
Allocated resources
```

---

# Step 6 - Check Node Status

Run:

```bash
kubectl get nodes
```

Expected:

```
Ready
```

---

# Step 7 - Fix Resource Request

Edit:

```bash
code yaml/lab-33/pending-pod.yaml
```

Change:

```yaml
memory: "100Gi"

cpu: "50"
```

to:

```yaml
memory: "128Mi"

cpu: "100m"
```

---

Delete old Pod:

```bash
kubectl delete pod pending-pod
```

---

Create again:

```bash
kubectl apply -f yaml/lab-33/pending-pod.yaml
```

---

# Step 8 - Verify

Run:

```bash
kubectl get pods
```

Expected:

```
pending-pod

Running
```

---

# Important Debug Commands

## Check Pods

```bash
kubectl get pods
```

---

## Describe Pod

```bash
kubectl describe pod POD_NAME
```

---

## Check Events

```bash
kubectl get events
```

---

## Check Nodes

```bash
kubectl get nodes
```

---

## Check Node Capacity

```bash
kubectl describe node NODE_NAME
```

---

# Verification

Run:

```bash
kubectl get pods
```

Pod should show:

```
Running
```

Check:

```bash
kubectl describe pod pending-pod
```

No scheduling errors should appear.

---

# Troubleshooting Checklist

When Pod is Pending:

1. Check Pod details

```bash
kubectl describe pod POD_NAME
```

2. Check Events

Look for:

```
FailedScheduling
```

3. Check resources

```bash
kubectl describe nodes
```

4. Check:

- CPU
- Memory
- Node labels
- Taints
- PVC

---

# Interview Questions

## 1. What does Pending mean in Kubernetes?

The Pod is created but Kubernetes cannot schedule it to a node.

---

## 2. How do you troubleshoot Pending Pods?

Use:

```bash
kubectl describe pod POD_NAME
```

Check Events.

---

## 3. Who assigns Pods to Nodes?

The Kubernetes Scheduler.

---

## 4. What causes FailedScheduling?

Examples:

- Not enough resources
- Node selector mismatch
- Taints
- Missing volumes

---

## 5. Difference between Pending and CrashLoopBackOff?

Pending:

Pod never starts.

CrashLoopBackOff:

Container starts but crashes.

---

# Commands Learned

```bash
kubectl get pods

kubectl describe pod

kubectl get events

kubectl describe nodes

kubectl get nodes
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod pending-pod
```

---

# Navigation

Previous:

[Lab 32 - ImagePullBackOff Troubleshooting](lab-32-imagepullbackoff.md)

Next:

[Lab 34 - Service Troubleshooting](lab-34-service-troubleshooting.md)

---

# Lab Completed ✅

You learned:

- Pending Pods
- Scheduler troubleshooting
- Resource problems
- Node scheduling issues