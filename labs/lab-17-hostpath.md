# LAB 17 - Kubernetes HostPath Volume

## Objective

In this lab, you will learn:

- What is HostPath volume
- How HostPath connects Pod storage with Node storage
- Mount a Node directory into a Pod
- Understand HostPath use cases
- Difference between HostPath and Persistent Volume

---

# What is HostPath?

HostPath allows a Pod to mount a file or directory from the Kubernetes Node filesystem.

Example:

```
Kubernetes Node

      |
      |
 /data/application

      |
      |
 Pod Container

      |
      |
 /app/data
```

The container can access files stored on the Node.

---

# Why Use HostPath?

Common use cases:

- Development environments
- Testing
- Logging agents
- Monitoring tools
- Accessing node-level files

---

# Important Note

HostPath directly uses Node storage.

If the Pod moves to another Node:

```
Node 1

/data/file.txt

     X

Pod moves

     |

Node 2

No file
```

The data may not exist.

For production applications, Persistent Volumes are preferred.

---

# HostPath vs Persistent Volume

| HostPath | Persistent Volume |
|---|---|
| Direct Node storage | Abstract storage layer |
| Node dependent | Can use cloud storage |
| Mostly testing | Production workloads |
| Simple setup | Managed storage |

---

# Architecture

```
          Kubernetes Cluster


              Node


        -----------------

        Host Directory

          /data/demo


              |

              |

             Pod


        Container Mount

          /app/data

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

# Step 1 - Create HostPath Pod YAML

Create:

```bash
touch yaml/lab-17/hostpath-pod.yaml
```

Open:

```bash
code yaml/lab-17/hostpath-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: hostpath-demo


spec:

  containers:

  - name: nginx

    image: nginx


    volumeMounts:

    - name: host-storage

      mountPath: /app/data



  volumes:

  - name: host-storage

    hostPath:

      path: /tmp/host-data

      type: DirectoryOrCreate
```

---

# Step 2 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-17/hostpath-pod.yaml
```

Expected:

```
pod/hostpath-demo created
```

---

# Step 3 - Verify Pod

Run:

```bash
kubectl get pods
```

Expected:

```
hostpath-demo Running
```

---

# Step 4 - Check Volume Details

Run:

```bash
kubectl describe pod hostpath-demo
```

Look for:

```
Volumes:

host-storage

Type:

HostPath
```

---

# Step 5 - Create File Inside Mounted Directory

Enter container:

```bash
kubectl exec -it hostpath-demo -- bash
```

Go to mount location:

```bash
cd /app/data
```

Create file:

```bash
echo "HostPath Storage Test" > test.txt
```

Check:

```bash
cat test.txt
```

Output:

```
HostPath Storage Test
```

Exit:

```bash
exit
```

---

# Step 6 - Restart Container Test

Delete Pod:

```bash
kubectl delete pod hostpath-demo
```

Create again:

```bash
kubectl apply -f yaml/lab-17/hostpath-pod.yaml
```

Check:

```bash
kubectl get pods
```

---

Enter again:

```bash
kubectl exec -it hostpath-demo -- bash
```

Check:

```bash
cat /app/data/test.txt
```

You should see:

```
HostPath Storage Test
```

---

# Verification

Run:

```bash
kubectl get pods
```

```bash
kubectl describe pod hostpath-demo
```

```bash
kubectl get pod hostpath-demo -o yaml
```

---

# Troubleshooting

## Pod cannot mount directory

Check:

```bash
kubectl describe pod hostpath-demo
```

Look at Events.

---

## Directory does not exist

Use:

```yaml
type: DirectoryOrCreate
```

Kubernetes will create it automatically.

---

# Interview Questions

## 1. What is HostPath?

HostPath mounts a file or directory from the Kubernetes Node into a Pod.

---

## 2. Is HostPath recommended for production?

Usually no.

Persistent Volumes are preferred.

---

## 3. What happens if Pod moves to another Node?

The Pod may not find the same data.

---

## 4. Difference between HostPath and PV?

HostPath directly uses Node storage.

PV provides an abstraction layer for storage.

---

## 5. Where is HostPath commonly used?

Testing, development, monitoring, and node-level applications.

---

# Commands Learned

```bash
kubectl get pods

kubectl describe pod

kubectl exec

kubectl apply -f

kubectl delete pod
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod hostpath-demo
```

---

# Navigation

Previous:

[Lab 16 - Persistent Volume](lab-16-persistent-volume.md)

Next:

[Lab 18 - Taints and Tolerations](lab-18-taints-tolerations.md)

---

# Lab Completed ✅

You learned:

- HostPath volume
- Node storage mounting
- HostPath limitations
- Difference between HostPath and PV