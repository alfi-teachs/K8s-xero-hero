# LAB 27 - Kubernetes Role and RoleBinding (RBAC)

## Objective

In this lab, you will learn:

- What RBAC is
- Difference between Authentication and Authorization
- What is a Role
- What is RoleBinding
- Create permissions for users/applications
- Test Kubernetes access control

---

# What is RBAC?

RBAC means:

```
Role-Based Access Control
```

It controls:

```
WHO can do WHAT on WHICH resource
```

Example:

A developer should:

```
View Pods
```

but should not:

```
Delete Deployments
```

RBAC controls this.

---

# Authentication vs Authorization

## Authentication

Question:

```
Who are you?
```

Example:

```
User login
ServiceAccount
```

---

## Authorization

Question:

```
What are you allowed to do?
```

Example:

```
Can this user create Pods?
```

---

# RBAC Flow

```
User / Application

        |

ServiceAccount

        |

RoleBinding

        |

Role

        |

Kubernetes API

```

---

# RBAC Components

Kubernetes RBAC has four main objects:

## 1. Role

Defines permissions inside a Namespace.

Example:

```
Allow reading Pods
```

---

## 2. RoleBinding

Connects:

```
User / ServiceAccount
        |
        |
      Role
```

---

## 3. ClusterRole

Permissions across the entire cluster.

---

## 4. ClusterRoleBinding

Connects users to ClusterRoles.

---

# Real World Example

Company has:

```
Developer Team
```

They need:

```
get pods
list pods
describe pods
```

They do not need:

```
delete nodes
create namespaces
```

Create a Role:

```
Pod Viewer Role
```

Bind it:

```
Developer User
        |
        |
Pod Viewer Role
```

---

# Lab Steps

# Step 1 - Check Existing Roles

Run:

```bash
kubectl get roles
```

---

# Step 2 - Create ServiceAccount

Create:

```bash
kubectl create serviceaccount developer-sa
```

Check:

```bash
kubectl get sa
```

---

# Step 3 - Create Role YAML

Create:

```bash
touch yaml/lab-27/pod-reader-role.yaml
```

Add:

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: Role


metadata:

  name: pod-reader


rules:

- apiGroups: [""]

  resources:

  - pods

  verbs:

  - get

  - list

  - watch
```

---

Apply:

```bash
kubectl apply -f yaml/lab-27/pod-reader-role.yaml
```

---

# Step 4 - Verify Role

Run:

```bash
kubectl get roles
```

Expected:

```
pod-reader
```

---

# Step 5 - Create RoleBinding

Create:

```bash
touch yaml/lab-27/rolebinding.yaml
```

Add:

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding


metadata:

  name: pod-reader-binding


subjects:

- kind: ServiceAccount

  name: developer-sa


roleRef:

  kind: Role

  name: pod-reader

  apiGroup: rbac.authorization.k8s.io
```

---

Apply:

```bash
kubectl apply -f yaml/lab-27/rolebinding.yaml
```

---

# Step 6 - Verify RoleBinding

Run:

```bash
kubectl get rolebindings
```

Expected:

```
pod-reader-binding
```

---

# Step 7 - Test Permission

Check if ServiceAccount can list Pods:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:developer-sa
```

Expected:

```
yes
```

---

Test Delete Permission:

```bash
kubectl auth can-i delete pods --as=system:serviceaccount:default:developer-sa
```

Expected:

```
no
```

Because we did not give delete permission.

---

# Step 8 - Check All Permissions

Run:

```bash
kubectl auth can-i --list --as=system:serviceaccount:default:developer-sa
```

---

# Verification

Check Role:

```bash
kubectl get role
```

Check RoleBinding:

```bash
kubectl get rolebinding
```

Check permissions:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:developer-sa
```

---

# Troubleshooting

## Permission Denied

Check:

```bash
kubectl describe role pod-reader
```

Check:

```bash
kubectl describe rolebinding pod-reader-binding
```

---

## RoleBinding Not Working

Verify ServiceAccount:

```bash
kubectl get sa
```

Verify subject:

```bash
kubectl get rolebinding pod-reader-binding -o yaml
```

---

# Interview Questions

## 1. What is RBAC?

RBAC controls access to Kubernetes resources based on roles.

---

## 2. Difference between Role and RoleBinding?

Role:

Defines permissions.

RoleBinding:

Assigns permissions to users or ServiceAccounts.

---

## 3. Difference between Role and ClusterRole?

Role:

Namespace level.

ClusterRole:

Cluster level.

---

## 4. What does kubectl auth can-i do?

It checks whether a user or ServiceAccount has permission to perform an action.

---

## 5. Does ServiceAccount give permissions?

No.

RoleBinding gives permissions.

---

# Commands Learned

```bash
kubectl get roles

kubectl get rolebindings

kubectl auth can-i

kubectl describe role

kubectl describe rolebinding
```

---

# Cleanup

Delete RoleBinding:

```bash
kubectl delete rolebinding pod-reader-binding
```

Delete Role:

```bash
kubectl delete role pod-reader
```

Delete ServiceAccount:

```bash
kubectl delete sa developer-sa
```

---

# Navigation

Previous:

[Lab 26 - ServiceAccount](lab-26-serviceaccount.md)

Next:

[Lab 28 - ClusterRole and ClusterRoleBinding](lab-28-clusterrole.md)

---

# Lab Completed ✅

You learned:

- RBAC concepts
- Role
- RoleBinding
- Kubernetes permissions
- Access control