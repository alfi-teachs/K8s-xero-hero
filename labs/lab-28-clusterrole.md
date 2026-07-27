# LAB 28 - Kubernetes ClusterRole and ClusterRoleBinding

## Objective

In this lab, you will learn:

- What is ClusterRole
- What is ClusterRoleBinding
- Difference between Role and ClusterRole
- Create cluster-level permissions
- Give access across namespaces
- Test Kubernetes authorization

---

# What is ClusterRole?

A ClusterRole defines permissions that apply across the entire Kubernetes cluster.

Unlike Role:

```
Role
 |
Namespace level
```

ClusterRole:

```
ClusterRole
 |
Entire Cluster
```

---

# Real World Example

A Kubernetes administrator needs access to:

```
All Pods

All Nodes

All Namespaces
```

A normal Role cannot do this.

We use:

```
ClusterRole
```

---

# Role vs ClusterRole

| Role | ClusterRole |
|------|-------------|
| Namespace level | Cluster level |
| Works inside one namespace | Works across namespaces |
| Used for Pods, Services | Used for Nodes, Namespaces |

---

# ClusterRole Examples

Common ClusterRoles:

```
cluster-admin

view

edit

admin
```

Check:

```bash
kubectl get clusterroles
```

---

# What is ClusterRoleBinding?

ClusterRoleBinding connects:

```
User / ServiceAccount

        |

ClusterRole

        |

Cluster Permissions
```

---

# RBAC Flow

```
User

 |

ServiceAccount

 |

ClusterRoleBinding

 |

ClusterRole

 |

Kubernetes API

```

---

# Lab Steps

# Step 1 - Check Existing ClusterRoles

Run:

```bash
kubectl get clusterroles
```

You will see:

```
cluster-admin

admin

edit

view
```

---

# Step 2 - Create ServiceAccount

Create:

```bash
kubectl create serviceaccount cluster-reader
```

Verify:

```bash
kubectl get sa
```

---

# Step 3 - Create ClusterRole

Create file:

```bash
touch yaml/lab-28/cluster-reader-role.yaml
```

Add:

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRole


metadata:

  name: pod-cluster-reader


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
kubectl apply -f yaml/lab-28/cluster-reader-role.yaml
```

---

# Step 4 - Verify ClusterRole

Run:

```bash
kubectl get clusterrole
```

Expected:

```
pod-cluster-reader
```

---

# Step 5 - Create ClusterRoleBinding

Create:

```bash
touch yaml/lab-28/clusterrolebinding.yaml
```

Add:

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRoleBinding


metadata:

  name: cluster-reader-binding


subjects:

- kind: ServiceAccount

  name: cluster-reader

  namespace: default


roleRef:

  kind: ClusterRole

  name: pod-cluster-reader

  apiGroup: rbac.authorization.k8s.io
```

---

Apply:

```bash
kubectl apply -f yaml/lab-28/clusterrolebinding.yaml
```

---

# Step 6 - Verify ClusterRoleBinding

Run:

```bash
kubectl get clusterrolebindings
```

Expected:

```
cluster-reader-binding
```

---

# Step 7 - Test Permission

Check if ServiceAccount can list Pods:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:cluster-reader
```

Expected:

```
yes
```

---

# Step 8 - Test Node Permission

Try:

```bash
kubectl auth can-i get nodes --as=system:serviceaccount:default:cluster-reader
```

Expected:

```
no
```

Because we only allowed Pods.

---

# Step 9 - View Permissions

Run:

```bash
kubectl auth can-i --list --as=system:serviceaccount:default:cluster-reader
```

---

# Verification

Check ClusterRole:

```bash
kubectl get clusterrole pod-cluster-reader
```

Check Binding:

```bash
kubectl get clusterrolebinding cluster-reader-binding
```

Check permissions:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:cluster-reader
```

---

# Troubleshooting

## Permission Denied

Check:

```bash
kubectl describe clusterrole pod-cluster-reader
```

Check:

```bash
kubectl describe clusterrolebinding cluster-reader-binding
```

---

## Wrong Namespace

Check ServiceAccount:

```bash
kubectl get sa
```

---

# Interview Questions

## 1. What is ClusterRole?

ClusterRole defines permissions across the Kubernetes cluster.

---

## 2. Difference between Role and ClusterRole?

Role:

Namespace-specific permissions.

ClusterRole:

Cluster-wide permissions.

---

## 3. What is ClusterRoleBinding?

It assigns ClusterRole permissions to users or ServiceAccounts.

---

## 4. Can ClusterRole be used with namespaces?

Yes.

A ClusterRole can give permissions to namespaced or cluster resources.

---

## 5. Which resource needs ClusterRole?

Examples:

```
Nodes

PersistentVolumes

Namespaces
```

---

# Commands Learned

```bash
kubectl get clusterroles

kubectl get clusterrolebindings

kubectl auth can-i

kubectl describe clusterrole

kubectl describe clusterrolebinding
```

---

# Cleanup

Delete ClusterRoleBinding:

```bash
kubectl delete clusterrolebinding cluster-reader-binding
```

Delete ClusterRole:

```bash
kubectl delete clusterrole pod-cluster-reader
```

Delete ServiceAccount:

```bash
kubectl delete sa cluster-reader
```

---

# Navigation

Previous:

[Lab 27 - Role and RoleBinding](lab-27-role-rolebinding.md)

Next:

[Lab 29 - Kubernetes Jobs](lab-29-job.md)

---

# Lab Completed ✅

You learned:

- ClusterRole
- ClusterRoleBinding
- Cluster-level permissions
- Advanced Kubernetes RBAC