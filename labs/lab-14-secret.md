# LAB 14 - Kubernetes Secret

## Objective

In this lab, you will learn:

- What is Kubernetes Secret
- Difference between ConfigMap and Secret
- Create Secret
- Encode and decode Secret data
- Use Secret as environment variables
- Mount Secret as files

---

# What is Kubernetes Secret?

A Kubernetes Secret is an object used to store sensitive information.

Examples:

- Passwords
- Usernames
- API keys
- Database credentials
- Tokens

---

# Real World Example

Application needs database connection.

Without Secret:

```
username=admin
password=password123
```

Problem:

Anyone viewing the application configuration can see the password.

---

With Secret:

```
Application

     |

 Kubernetes Secret

     |

 Username + Password
```

The application reads the value when it runs.

---

# ConfigMap vs Secret

| ConfigMap | Secret |
|---|---|
| Normal configuration | Sensitive information |
| Application settings | Passwords |
| URLs | API keys |
| Plain text | Base64 encoded |

---

# Secret Architecture

```
              Secret

                |

             Kubernetes Pod

                |

            Container
```

---

# Prerequisites

Check Kubernetes:

```bash
kubectl get nodes
```

Expected:

```
minikube Ready
```

---

# Lab Steps

# Step 1 - Create Secret Using YAML

Create file:

```bash
touch yaml/lab-14/database-secret.yaml
```

Open:

```bash
code yaml/lab-14/database-secret.yaml
```

Add:

```yaml
apiVersion: v1

kind: Secret

metadata:

  name: database-secret


type: Opaque


data:

  username: YWRtaW4=

  password: cGFzc3dvcmQxMjM=
```

---

# Understanding Base64

Kubernetes Secrets store values in Base64 format.

Example:

Username:

```
admin
```

Encoded:

```
YWRtaW4=
```

Password:

```
password123
```

Encoded:

```
cGFzc3dvcmQxMjM=
```

---

# Step 2 - Create Secret

Run:

```bash
kubectl apply -f yaml/lab-14/database-secret.yaml
```

Expected:

```
secret/database-secret created
```

---

# Step 3 - Verify Secret

Check:

```bash
kubectl get secrets
```

Expected:

```
database-secret
```

---

Describe:

```bash
kubectl describe secret database-secret
```

Notice:

```
Data
====
username
password
```

---

# Step 4 - View Secret YAML

Run:

```bash
kubectl get secret database-secret -o yaml
```

You will see:

```yaml
username: YWRtaW4=
password: cGFzc3dvcmQxMjM=
```

---

# Step 5 - Decode Secret Values

Decode username:

```bash
echo YWRtaW4= | base64 --decode
```

Output:

```
admin
```

---

Decode password:

```bash
echo cGFzc3dvcmQxMjM= | base64 --decode
```

Output:

```
password123
```

---

# Step 6 - Create Pod Using Secret

Create:

```bash
touch yaml/lab-14/secret-pod.yaml
```

Open:

```bash
code yaml/lab-14/secret-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: secret-demo


spec:

  containers:

  - name: nginx

    image: nginx


    env:

    - name: DB_USERNAME

      valueFrom:

        secretKeyRef:

          name: database-secret

          key: username


    - name: DB_PASSWORD

      valueFrom:

        secretKeyRef:

          name: database-secret

          key: password
```

---

# Step 7 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-14/secret-pod.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
secret-demo Running
```

---

# Step 8 - Verify Secret Inside Pod

Enter container:

```bash
kubectl exec -it secret-demo -- bash
```

Check variables:

```bash
env | grep DB
```

Output:

```
DB_USERNAME=admin
DB_PASSWORD=password123
```

Exit:

```bash
exit
```

---

# Step 9 - Mount Secret as Files

Create:

```bash
touch yaml/lab-14/secret-volume.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: secret-volume-demo


spec:

  containers:

  - name: nginx

    image: nginx


    volumeMounts:

    - name: secret-volume

      mountPath: /secret


  volumes:

  - name: secret-volume

    secret:

      secretName: database-secret
```

---

Create:

```bash
kubectl apply -f yaml/lab-14/secret-volume.yaml
```

---

Enter:

```bash
kubectl exec -it secret-volume-demo -- bash
```

Check:

```bash
ls /secret
```

Output:

```
password
username
```

Exit:

```bash
exit
```

---

# Verification

Run:

```bash
kubectl get secrets
```

```bash
kubectl get pods
```

```bash
kubectl describe secret database-secret
```

---

# Troubleshooting

## Secret not found

Check:

```bash
kubectl get secrets
```

Verify name:

```yaml
secretName: database-secret
```

---

## Cannot decode secret

Make sure Base64 command is correct:

Linux:

```bash
echo value | base64 --decode
```

Windows PowerShell:

```powershell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("value"))
```

---

# Interview Questions

## 1. What is Kubernetes Secret?

Secret stores sensitive information like passwords and tokens.

---

## 2. Difference between Secret and ConfigMap?

ConfigMap stores normal configuration.

Secret stores sensitive data.

---

## 3. Are Kubernetes Secrets encrypted?

By default, Secret data is Base64 encoded, not encrypted.

---

## 4. How can Pods access Secrets?

Two ways:

1. Environment variables

2. Mounted volumes

---

## 5. What is Secret type Opaque?

Opaque is the default Secret type for arbitrary user-defined data.

---

# Commands Learned

```bash
kubectl get secrets

kubectl describe secret

kubectl get secret -o yaml

kubectl apply -f file.yaml

kubectl exec
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod secret-demo
```

```bash
kubectl delete pod secret-volume-demo
```

Delete Secret:

```bash
kubectl delete secret database-secret
```

---

# Navigation

Previous:

[Lab 13 - ConfigMap](lab-13-configmap.md)

Next:

[Lab 15 - EmptyDir Volume](lab-15-emptydir.md)

---

# Lab Completed ✅

You learned:

- Kubernetes Secret
- Base64 encoding
- Secret as environment variables
- Secret as volume files
- Secure configuration management