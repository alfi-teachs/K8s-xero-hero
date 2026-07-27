# LAB 32 - ImagePullBackOff Troubleshooting

## Objective

In this lab, you will learn:

- What ImagePullBackOff means
- Why Kubernetes cannot pull images
- Common image pull problems
- How to troubleshoot image failures
- How to fix incorrect image configurations

---

# What is ImagePullBackOff?

ImagePullBackOff means:

```
Kubernetes tried to download container image

        |

Image download failed

        |

Kubernetes waits

        |

Retries again
```

The Pod cannot start because the container image is unavailable.

---

# Container Image Flow

```
Pod YAML

   |

Image name

   |

Container Registry

   |

Download Image

   |

Start Container

```

---

# Common Causes

## 1. Wrong Image Name

Example:

Wrong:

```yaml
image: nginx12345
```

Kubernetes searches:

```
Docker Hub
```

but image does not exist.

---

## 2. Wrong Image Tag

Example:

```yaml
image: nginx:abc
```

Tag does not exist.

---

## 3. Private Registry

Image exists but requires authentication.

Example:

```
AWS ECR
Docker Private Registry
Azure Container Registry
```

---

## 4. Network Problem

Worker node cannot reach registry.

---

## 5. Image Deleted

Registry image no longer exists.

---

# Troubleshooting Flow

```
Pod Not Starting

       |

kubectl get pods

       |

ImagePullBackOff

       |

kubectl describe pod

       |

Check Events

       |

Fix Image

```

---

# Lab Steps

# Step 1 - Create Broken Image Pod

Create:

```bash
touch yaml/lab-32/wrong-image-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: wrong-image-pod


spec:

  containers:

  - name: nginx

    image: nginx-not-exist-12345

```

---

# Step 2 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-32/wrong-image-pod.yaml
```

---

# Step 3 - Check Pod Status

Run:

```bash
kubectl get pods
```

Expected:

```
wrong-image-pod

ImagePullBackOff
```

---

# Step 4 - Describe Pod

Run:

```bash
kubectl describe pod wrong-image-pod
```

Look at:

```
Events
```

Example:

```
Failed to pull image

repository does not exist
```

---

# Step 5 - Check Image Name

Run:

```bash
kubectl get pod wrong-image-pod -o yaml
```

Look for:

```yaml
image:
```

---

# Step 6 - Fix Image

Edit:

```bash
code yaml/lab-32/wrong-image-pod.yaml
```

Change:

```yaml
image: nginx-not-exist-12345
```

to:

```yaml
image: nginx
```

---

Delete old Pod:

```bash
kubectl delete pod wrong-image-pod
```

Create again:

```bash
kubectl apply -f yaml/lab-32/wrong-image-pod.yaml
```

---

# Step 7 - Verify Pod

Check:

```bash
kubectl get pods
```

Expected:

```
wrong-image-pod

Running
```

---

# Step 8 - Check Image Details

Run:

```bash
kubectl describe pod wrong-image-pod
```

Look for:

```
Image: nginx
```

---

# Private Registry Example

For private images:

```
Private Registry

        |

ImagePullSecret

        |

Pod

```

Create secret:

```bash
kubectl create secret docker-registry myregistrykey
```

Use in Pod:

```yaml
imagePullSecrets:

- name: myregistrykey
```

---

# Important Debug Commands

## Check Pod Status

```bash
kubectl get pods
```

---

## Detailed Error

```bash
kubectl describe pod POD_NAME
```

---

## Check Events

```bash
kubectl get events
```

---

## Check YAML

```bash
kubectl get pod POD_NAME -o yaml
```

---

# Verification

Run:

```bash
kubectl get pods
```

Expected:

```
Running
```

Check:

```bash
kubectl describe pod wrong-image-pod
```

No image pull errors should appear.

---

# Troubleshooting Checklist

When you see ImagePullBackOff:

1. Check image name

```
Is image correct?
```

2. Check tag

```
Does tag exist?
```

3. Check registry access

```
Can node reach registry?
```

4. Check authentication

```
Does private registry need secret?
```

---

# Interview Questions

## 1. What is ImagePullBackOff?

It means Kubernetes cannot download the container image.

---

## 2. How do you troubleshoot ImagePullBackOff?

Use:

```bash
kubectl describe pod
```

Check Events.

---

## 3. Difference between ImagePullBackOff and CrashLoopBackOff?

ImagePullBackOff:

Image cannot be downloaded.

CrashLoopBackOff:

Container starts but crashes.

---

## 4. How do you access private images?

Using:

```
imagePullSecrets
```

---

## 5. Where does Kubernetes pull images from?

Usually:

```
Docker Hub
```

or private container registries.

---

# Commands Learned

```bash
kubectl get pods

kubectl describe pod

kubectl get events

kubectl get pod -o yaml
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod wrong-image-pod
```

---

# Navigation

Previous:

[Lab 31 - CrashLoopBackOff Troubleshooting](lab-31-crashloopbackoff.md)

Next:

[Lab 33 - Pending Pods Troubleshooting](lab-33-pending-pods.md)

---

# Lab Completed ✅

You learned:

- ImagePullBackOff
- Image troubleshooting
- Registry problems
- Image pull failures