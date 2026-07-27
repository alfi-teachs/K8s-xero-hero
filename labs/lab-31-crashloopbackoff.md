# LAB 31 - CrashLoopBackOff Troubleshooting

## Objective

In this lab, you will learn:

- What CrashLoopBackOff means
- Common reasons for Pod failure
- How to check Pod status
- How to use kubectl describe
- How to check container logs
- How to fix application crashes

---

# What is CrashLoopBackOff?

CrashLoopBackOff means:

```
Container starts
      |
      |
Application crashes
      |
      |
Kubernetes restarts container
      |
      |
Container crashes again
      |
      |
Kubernetes waits before restarting
```

Kubernetes keeps trying to restart the failed container.

---

# Real World Example

Imagine:

Your application starts:

```
Application running
```

Then:

```
Database connection failed
```

Application exits.

Kubernetes says:

```
I will restart it
```

But it fails again.

Result:

```
CrashLoopBackOff
```

---

# Common Causes

## 1. Application Error

Example:

```
Application configuration mistake
```

---

## 2. Wrong Environment Variables

Example:

```
Database URL missing
```

---

## 3. Wrong Command

Example:

```yaml
command:
- wrong-command
```

Container exits immediately.

---

## 4. Missing Configuration

Example:

```
ConfigMap missing
Secret missing
```

---

## 5. Resource Problems

Example:

```
Out Of Memory
```

---

# Troubleshooting Flow

```
Pod Failed

    |

kubectl get pods

    |

kubectl describe pod

    |

kubectl logs

    |

Find Error

    |

Fix YAML/Application

```

---

# Lab Steps

# Step 1 - Create Broken Pod

Create:

```bash
touch yaml/lab-31/crash-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: crash-pod


spec:

  containers:

  - name: crash-container

    image: busybox

    command:

    - sh

    - -c

    - exit 1
```

---

# Step 2 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-31/crash-pod.yaml
```

---

# Step 3 - Check Pod Status

Run:

```bash
kubectl get pods
```

Expected:

```
crash-pod

CrashLoopBackOff
```

---

# Step 4 - Describe Pod

Run:

```bash
kubectl describe pod crash-pod
```

Look for:

```
Events
```

Example:

```
Back-off restarting failed container
```

---

# Step 5 - Check Logs

Run:

```bash
kubectl logs crash-pod
```

You may see:

```
Error
```

---

# Step 6 - Check Previous Container Logs

When a container restarts:

Run:

```bash
kubectl logs crash-pod --previous
```

This shows logs from the previous crashed container.

---

# Step 7 - Fix the Pod

Delete failed Pod:

```bash
kubectl delete pod crash-pod
```

Edit YAML:

```bash
code yaml/lab-31/crash-pod.yaml
```

Change:

```yaml
- exit 1
```

to:

```yaml
- echo "Application Started"
```

---

Create again:

```bash
kubectl apply -f yaml/lab-31/crash-pod.yaml
```

---

Check:

```bash
kubectl get pods
```

---

# Step 8 - Check Logs Again

Run:

```bash
kubectl logs crash-pod
```

Output:

```
Application Started
```

---

# Important Debug Commands

## Check Pods

```bash
kubectl get pods
```

---

## Detailed Pod Information

```bash
kubectl describe pod POD_NAME
```

---

## View Logs

```bash
kubectl logs POD_NAME
```

---

## Previous Crash Logs

```bash
kubectl logs POD_NAME --previous
```

---

## Check Events

```bash
kubectl get events
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
kubectl logs crash-pod
```

---

# Troubleshooting Checklist

When you see CrashLoopBackOff:

1. Check Pod status

```bash
kubectl get pods
```

2. Describe Pod

```bash
kubectl describe pod POD_NAME
```

3. Check logs

```bash
kubectl logs POD_NAME
```

4. Check previous logs

```bash
kubectl logs POD_NAME --previous
```

5. Check configuration

- Image
- Command
- Environment variables
- Secrets
- ConfigMaps

---

# Interview Questions

## 1. What is CrashLoopBackOff?

It means a container is repeatedly crashing and Kubernetes is waiting before restarting it.

---

## 2. How do you troubleshoot CrashLoopBackOff?

Use:

```bash
kubectl describe pod

kubectl logs

kubectl logs --previous
```

---

## 3. Why does Kubernetes restart containers?

To maintain the desired state.

---

## 4. Difference between Error and CrashLoopBackOff?

Error:

Container failed once.

CrashLoopBackOff:

Container keeps failing repeatedly.

---

## 5. What command shows Pod events?

```bash
kubectl describe pod
```

or

```bash
kubectl get events
```

---

# Commands Learned

```bash
kubectl get pods

kubectl describe pod

kubectl logs

kubectl logs --previous

kubectl get events
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod crash-pod
```

---

# Navigation

Previous:

[Lab 30 - Kubernetes CronJob](lab-30-cronjob.md)

Next:

[Lab 32 - ImagePullBackOff Troubleshooting](lab-32-imagepullbackoff.md)

---

# Lab Completed ✅

You learned:

- CrashLoopBackOff
- Pod debugging
- Container logs
- Kubernetes troubleshooting workflow