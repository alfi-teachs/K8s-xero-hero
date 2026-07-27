# LAB 26 - Kubernetes ServiceAccount

## Objective

In this lab, you will learn:

- What is a ServiceAccount
- Why Pods need identity
- Create a ServiceAccount
- Attach ServiceAccount to Pods
- Verify ServiceAccount usage
- Understand Kubernetes authentication

---

# What is a ServiceAccount?

A ServiceAccount provides an identity for applications running inside Kubernetes.

Humans use:

```
User Account
```

Applications use:

```
ServiceAccount
```

Example:

A Pod needs permission to read Kubernetes resources.

Instead of giving permission to everyone:

```
Pod
 |
ServiceAccount
 |
RBAC Permission
 |
Kubernetes API
```

---

# Real World Example

Imagine an application:

```
Monitoring Application
```

It needs to read:

```
Pods
Nodes
Services
```

Kubernetes asks:

"Who is making this request?"

The answer:

```
ServiceAccount
```

---

# Authentication Flow

```
Application Pod

       |

ServiceAccount

       |

API Server

       |

RBAC Check

       |

Allow / Deny

```

---

# Default ServiceAccount

Every Namespace automatically gets a default ServiceAccount.

Check:

```bash
kubectl get serviceaccounts
```

Example:

```
NAME
default
```

---

# ServiceAccount Components

A ServiceAccount provides:

- Identity
- Authentication
- Access control connection

It does NOT provide permissions.

Permissions are given by:

```
Role

RoleBinding

ClusterRole

ClusterRoleBinding
```

---

# Lab Steps

# Step 1 - Check Existing ServiceAccounts

Run:

```bash
kubectl get serviceaccounts
```

Output:

```
NAME
default
```

---

# Step 2 - Create ServiceAccount Using Command

Run:

```bash
kubectl create serviceaccount app-sa
```

Expected:

```
serviceaccount/app-sa created
```

---

# Step 3 - Verify ServiceAccount

Run:

```bash
kubectl get serviceaccounts
```

Expected:

```
NAME

default

app-sa
```

---

# Step 4 - Create ServiceAccount Using YAML

Create:

```bash
touch yaml/lab-26/serviceaccount.yaml
```

Add:

```yaml
apiVersion: v1

kind: ServiceAccount


metadata:

  name: nginx-sa
```

---

Apply:

```bash
kubectl apply -f yaml/lab-26/serviceaccount.yaml
```

---

# Step 5 - Create Pod With ServiceAccount

Create:

```bash
touch yaml/lab-26/pod-serviceaccount.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: nginx-sa-pod


spec:

  serviceAccountName: nginx-sa


  containers:

  - name: nginx

    image: nginx

    ports:

    - containerPort: 80
```

---

# Step 6 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-26/pod-serviceaccount.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
nginx-sa-pod   Running
```

---

# Step 7 - Verify ServiceAccount Attached

Run:

```bash
kubectl describe pod nginx-sa-pod
```

Look for:

```
Service Account:
nginx-sa
```

---

# Step 8 - Check Pod ServiceAccount

Run:

```bash
kubectl get pod nginx-sa-pod -o yaml
```

Look for:

```yaml
serviceAccountName: nginx-sa
```

---

# Step 9 - View Token Inside Pod

Enter Pod:

```bash
kubectl exec -it nginx-sa-pod -- /bin/bash
```

Check:

```bash
ls /var/run/secrets/kubernetes.io/serviceaccount/
```

You will see:

```
ca.crt
namespace
token
```

Exit:

```bash
exit
```

---

# Step 10 - Delete ServiceAccount

Delete Pod:

```bash
kubectl delete pod nginx-sa-pod
```

Delete ServiceAccounts:

```bash
kubectl delete serviceaccount app-sa
```

```bash
kubectl delete serviceaccount nginx-sa
```

---

# Verification

Check ServiceAccounts:

```bash
kubectl get sa
```

Check Pod:

```bash
kubectl describe pod nginx-sa-pod
```

Check YAML:

```bash
kubectl get pod nginx-sa-pod -o yaml
```

---

# Troubleshooting

## Pod Cannot Access Kubernetes API

Check:

```bash
kubectl describe pod POD_NAME
```

Verify:

```
serviceAccountName
```

---

## ServiceAccount Not Found

Check:

```bash
kubectl get sa
```

Create again:

```bash
kubectl create serviceaccount NAME
```

---

# Interview Questions

## 1. What is a ServiceAccount?

A ServiceAccount provides identity for applications running inside Kubernetes.

---

## 2. Difference between User Account and ServiceAccount?

User Account:

Used by humans.

ServiceAccount:

Used by applications.

---

## 3. Does ServiceAccount provide permissions?

No.

Permissions are provided using RBAC.

---

## 4. How does a Pod use a ServiceAccount?

Using:

```yaml
serviceAccountName:
```

---

## 5. Where is ServiceAccount token stored?

Inside:

```
/var/run/secrets/kubernetes.io/serviceaccount/
```

---

# Commands Learned

```bash
kubectl get serviceaccounts

kubectl create serviceaccount

kubectl describe pod

kubectl get pod -o yaml

kubectl exec
```

---

# Cleanup

Delete Pod:

```bash
kubectl delete pod nginx-sa-pod
```

Delete ServiceAccounts:

```bash
kubectl delete sa nginx-sa
```

```bash
kubectl delete sa app-sa
```

---

# Navigation

Previous:

[Lab 25 - Horizontal Pod Autoscaler](lab-25-hpa.md)

Next:

[Lab 27 - Role and RoleBinding](lab-27-role-rolebinding.md)

---

# Lab Completed ✅

You learned:

- ServiceAccount
- Pod identity
- Kubernetes authentication
- Application identity management