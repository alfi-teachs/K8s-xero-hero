# LAB 15 - Kubernetes EmptyDir Volume

## Objective

In this lab, you will learn:

- What is Kubernetes Volume
- What is EmptyDir volume
- How containers share storage
- Temporary storage inside Pods
- Volume mounting

---

# What is Kubernetes Volume?

A Volume is a storage location attached to a Pod.

Containers are temporary.

If a container restarts:

- Container data can disappear
- Volume data can survive depending on volume type

---

# Why Do We Need Volumes?

Example:

Application:

```
Frontend Container

       |

Backend Container

       |

Shared Files
```

Both containers need access to the same data.

Volumes solve this problem.

---

# What is EmptyDir?

EmptyDir is a temporary storage volume created when a Pod starts.

Important points:

- Created automatically by Kubernetes
- Exists as long as Pod exists
- Deleted when Pod is deleted
- Used for temporary data sharing

---

# EmptyDir Lifecycle

```
Pod Created

     |

EmptyDir Created

     |

Containers Use Storage

     |

Pod Deleted

     |

EmptyDir Deleted
```

---

# Real World Example

Application:

```
Container 1

Generates files

      |

      |

EmptyDir Volume

      |

      |

Container 2

Reads files
```

---

# Architecture

```
             Pod

    -------------------

    |                 |

Container 1     Container 2

    |                 |

    -------------------

          |

      EmptyDir Volume
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

# Step 1 - Create EmptyDir Pod

Create:

```bash
touch yaml/lab-15/emptydir-pod.yaml
```

Open:

```bash
code yaml/lab-15/emptydir-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: emptydir-demo


spec:

  containers:

  - name: writer-container

    image: busybox


    command:

    - sh

    - -c

    - "echo Kubernetes EmptyDir Lab > /data/test.txt && sleep 3600"


    volumeMounts:

    - name: shared-storage

      mountPath: /data



  - name: reader-container

    image: busybox


    command:

    - sh

    - -c

    - "sleep 3600"


    volumeMounts:

    - name: shared-storage

      mountPath: /data



  volumes:

  - name: shared-storage

    emptyDir: {}
```

---

# Step 2 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-15/emptydir-pod.yaml
```

Expected:

```
pod/emptydir-demo created
```

---

# Step 3 - Verify Pod

Run:

```bash
kubectl get pods
```

Expected:

```
NAME             READY
emptydir-demo    2/2
```

Notice:

Two containers are running inside one Pod.

---

# Step 4 - Check Pod Details

Run:

```bash
kubectl describe pod emptydir-demo
```

Find:

```
Volumes:

shared-storage

Type:

EmptyDir
```

---

# Step 5 - Check Data From Writer Container

Enter writer container:

```bash
kubectl exec -it emptydir-demo -c writer-container -- sh
```

Check:

```bash
ls /data
```

Output:

```
test.txt
```

View file:

```bash
cat /data/test.txt
```

Output:

```
Kubernetes EmptyDir Lab
```

Exit:

```bash
exit
```

---

# Step 6 - Read Same Data From Second Container

Enter reader container:

```bash
kubectl exec -it emptydir-demo -c reader-container -- sh
```

Check:

```bash
ls /data
```

Output:

```
test.txt
```

Read:

```bash
cat /data/test.txt
```

Output:

```
Kubernetes EmptyDir Lab
```

Exit:

```bash
exit
```

---

# Step 7 - Delete Pod

Delete:

```bash
kubectl delete pod emptydir-demo
```

Check:

```bash
kubectl get pods
```

---

# Step 8 - Recreate Pod

Run:

```bash
kubectl apply -f yaml/lab-15/emptydir-pod.yaml
```

Enter container:

```bash
kubectl exec -it emptydir-demo -c reader-container -- sh
```

Check:

```bash
ls /data
```

You will notice:

```
No files
```

Because EmptyDir storage was deleted when the Pod was removed.

---

# Verification

Run:

```bash
kubectl get pods
```

```bash
kubectl describe pod emptydir-demo
```

```bash
kubectl get pod emptydir-demo -o yaml
```

---

# Troubleshooting

## Pod not starting

Check:

```bash
kubectl describe pod emptydir-demo
```

Check events.

---

## Container cannot see files

Verify both containers have:

```yaml
volumeMounts:
- name: shared-storage
  mountPath: /data
```

---

# Interview Questions

## 1. What is EmptyDir?

EmptyDir is temporary storage created when a Pod starts.

---

## 2. When is EmptyDir deleted?

When the Pod is deleted.

---

## 3. Can multiple containers share EmptyDir?

Yes.

Containers in the same Pod can share the same volume.

---

## 4. Is EmptyDir persistent?

No.

It is temporary storage.

---

## 5. Where is EmptyDir stored?

By default:

- Node filesystem
- Memory if medium is set to Memory

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
kubectl delete pod emptydir-demo
```

---

# Navigation

Previous:

[Lab 14 - Secret](lab-14-secret.md)

Next:

[Lab 16 - Persistent Volume](lab-16-persistent-volume.md)

---

# Lab Completed ✅

You learned:

- Kubernetes volumes
- EmptyDir storage
- Container communication
- Shared storage between containers