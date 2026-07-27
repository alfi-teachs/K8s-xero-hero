# LAB 16 - Kubernetes Persistent Volume (PV) and Persistent Volume Claim (PVC)

## Objective

In this lab, you will learn:

- What is Persistent Volume
- What is Persistent Volume Claim
- Difference between EmptyDir and Persistent Storage
- Create PV
- Create PVC
- Attach storage to Pod
- Verify persistent data

---

# Why Do We Need Persistent Storage?

Containers are temporary.

Example:

```
Pod

 |
 |
Container

 |
 |
Application Data
```

If the Pod is deleted:

```
Pod Deleted

      |

Container Deleted

      |

Data Lost
```

For databases and applications, we need storage that survives Pod deletion.

---

# What is Persistent Volume (PV)?

Persistent Volume is storage created by the Kubernetes administrator.

PV represents actual storage.

Examples:

- AWS EBS
- Azure Disk
- Google Persistent Disk
- NFS
- Local Storage

---

# What is Persistent Volume Claim (PVC)?

PVC is a request for storage made by an application.

Think:

PV = Available storage

PVC = Application asking for storage

---

# PV and PVC Relationship

```
Administrator

      |

      |

Persistent Volume

      |

      |

Persistent Volume Claim

      |

      |

Pod

      |

      |

Container
```

---

# Real World Example

Database needs:

```
50GB Storage
```

Developer creates:

```
PVC request: 50GB
```

Kubernetes finds matching:

```
PV: 50GB
```

Then connects:

```
Database Pod

      |

     PVC

      |

     PV
```

---

# EmptyDir vs Persistent Volume

| EmptyDir | Persistent Volume |
|-|-|
| Temporary | Permanent storage |
| Deleted with Pod | Survives Pod deletion |
| Testing purpose | Production workloads |
| Node storage | External storage |

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

# Step 1 - Create Persistent Volume

Create:

```bash
touch yaml/lab-16/pv.yaml
```

Open:

```bash
code yaml/lab-16/pv.yaml
```

Add:

```yaml
apiVersion: v1

kind: PersistentVolume


metadata:

  name: local-pv


spec:

  capacity:

    storage: 1Gi


  accessModes:

    - ReadWriteOnce


  hostPath:

    path: /data/pv
```

---

# Understanding PV YAML

## capacity

Storage size:

```
1Gi
```

---

## accessModes

Defines how storage can be accessed.

Example:

```
ReadWriteOnce
```

Means:

One node can mount storage as read/write.

---

## hostPath

Uses local node storage.

For Minikube:

```
Node filesystem
```

---

# Step 2 - Create PV

Run:

```bash
kubectl apply -f yaml/lab-16/pv.yaml
```

Expected:

```
persistentvolume/local-pv created
```

---

# Step 3 - Verify PV

Run:

```bash
kubectl get pv
```

Expected:

```
NAME        CAPACITY
local-pv    1Gi
```

---

# Step 4 - Create PVC

Create:

```bash
touch yaml/lab-16/pvc.yaml
```

Open:

```bash
code yaml/lab-16/pvc.yaml
```

Add:

```yaml
apiVersion: v1

kind: PersistentVolumeClaim


metadata:

  name: local-pvc


spec:

  accessModes:

    - ReadWriteOnce


  resources:

    requests:

      storage: 1Gi
```

---

# Step 5 - Create PVC

Run:

```bash
kubectl apply -f yaml/lab-16/pvc.yaml
```

---

Check:

```bash
kubectl get pvc
```

Expected:

```
NAME
local-pvc
```

---

# Step 6 - Check PV Binding

Run:

```bash
kubectl get pv
```

Before:

```
Available
```

After PVC:

```
Bound
```

---

# Step 7 - Create Pod Using PVC

Create:

```bash
touch yaml/lab-16/storage-pod.yaml
```

Open:

```bash
code yaml/lab-16/storage-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: storage-demo


spec:

  containers:

  - name: nginx

    image: nginx


    volumeMounts:

    - name: storage-volume

      mountPath: /data


  volumes:

  - name: storage-volume

    persistentVolumeClaim:

      claimName: local-pvc
```

---

# Step 8 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-16/storage-pod.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
storage-demo Running
```

---

# Step 9 - Test Persistent Storage

Enter Pod:

```bash
kubectl exec -it storage-demo -- bash
```

Create file:

```bash
echo "Persistent Storage Test" > /data/test.txt
```

Check:

```bash
cat /data/test.txt
```

Output:

```
Persistent Storage Test
```

Exit:

```bash
exit
```

---

# Step 10 - Delete Pod

Delete:

```bash
kubectl delete pod storage-demo
```

---

Create Pod again:

```bash
kubectl apply -f yaml/lab-16/storage-pod.yaml
```

---

Check data:

```bash
kubectl exec -it storage-demo -- bash
```

Run:

```bash
cat /data/test.txt
```

Output:

```
Persistent Storage Test
```

Data still exists.

---

# Verification

Run:

```bash
kubectl get pv
```

```bash
kubectl get pvc
```

```bash
kubectl get pods
```

---

# Troubleshooting

## PVC stuck in Pending

Check:

```bash
kubectl describe pvc local-pvc
```

Possible reasons:

- No matching PV
- Wrong storage size
- Wrong access mode

---

## Pod cannot mount volume

Check:

```bash
kubectl describe pod storage-demo
```

---

# Interview Questions

## 1. What is Persistent Volume?

PV is storage resource available in Kubernetes.

---

## 2. What is PVC?

PVC is a request for storage by an application.

---

## 3. Difference between PV and PVC?

PV:

Storage provider resource.

PVC:

Application storage request.

---

## 4. Why use PVC?

To separate storage management from application deployment.

---

## 5. Does data survive Pod deletion?

Yes, when using persistent storage.

---

# Commands Learned

```bash
kubectl get pv

kubectl get pvc

kubectl describe pv

kubectl describe pvc

kubectl apply -f

kubectl delete pod
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod storage-demo
```

Delete PVC:

```bash
kubectl delete pvc local-pvc
```

Delete PV:

```bash
kubectl delete pv local-pv
```

---

# Navigation

Previous:

[Lab 15 - EmptyDir Volume](lab-15-emptydir.md)

Next:

[Lab 17 - HostPath Volume](lab-17-hostpath.md)

---

# Lab Completed ✅

You learned:

- Persistent Volume
- Persistent Volume Claim
- Storage lifecycle
- Mounting persistent storage into Pods