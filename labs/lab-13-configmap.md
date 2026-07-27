# LAB 13 - Kubernetes ConfigMap

## Objective

In this lab, you will learn:

- What is ConfigMap
- Why ConfigMap is required
- Create ConfigMap
- Store configuration data
- Use ConfigMap as environment variables
- Mount ConfigMap as files
- Separate configuration from application

---

# What is ConfigMap?

ConfigMap is a Kubernetes object used to store non-sensitive configuration data.

Examples:

- Application settings
- Database host names
- Environment values
- Configuration files

---

# Real World Example

Without ConfigMap:

Application code:

```
database_url=production-db.com
```

Problem:

Every environment needs different values.

Development:

```
database_url=dev-db.com
```

Production:

```
database_url=prod-db.com
```

---

With ConfigMap:

Application:

```
Read configuration
        |
        |
    ConfigMap
        |
        |
Different values per environment
```

---

# ConfigMap vs Secret

| ConfigMap | Secret |
|---|---|
| Normal configuration | Sensitive data |
| Database host | Password |
| Application settings | API keys |
| Not encrypted | Encoded |

---

# ConfigMap Architecture

```
              ConfigMap

                  |

             Kubernetes Pod

                  |

             Container
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

# Step 1 - Create ConfigMap

Create:

```bash
touch yaml/lab-13/app-configmap.yaml
```

Open:

```bash
code yaml/lab-13/app-configmap.yaml
```

Add:

```yaml
apiVersion: v1

kind: ConfigMap

metadata:
  name: app-config


data:

  APP_NAME: Kubernetes-App

  APP_ENV: Development

  APP_VERSION: "1.0"
```

---

# Step 2 - Create ConfigMap

Run:

```bash
kubectl apply -f yaml/lab-13/app-configmap.yaml
```

Expected:

```
configmap/app-config created
```

---

# Step 3 - Verify ConfigMap

Run:

```bash
kubectl get configmap
```

Expected:

```
app-config
```

---

View details:

```bash
kubectl describe configmap app-config
```

---

View YAML:

```bash
kubectl get configmap app-config -o yaml
```

---

# Step 4 - Create Pod Using ConfigMap

Create:

```bash
touch yaml/lab-13/configmap-pod.yaml
```

Open:

```bash
code yaml/lab-13/configmap-pod.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: configmap-demo


spec:

  containers:

  - name: nginx

    image: nginx


    env:

    - name: APP_NAME

      valueFrom:

        configMapKeyRef:

          name: app-config

          key: APP_NAME


    - name: APP_ENV

      valueFrom:

        configMapKeyRef:

          name: app-config

          key: APP_ENV
```

---

# Step 5 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-13/configmap-pod.yaml
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
configmap-demo Running
```

---

# Step 6 - Check Environment Variables

Enter Pod:

```bash
kubectl exec -it configmap-demo -- bash
```

Inside container:

```bash
env
```

Look for:

```
APP_NAME=Kubernetes-App

APP_ENV=Development
```

Exit:

```bash
exit
```

---

# Step 7 - Mount ConfigMap as File

Create:

```bash
touch yaml/lab-13/configmap-volume.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: configmap-volume-demo


spec:

  containers:

  - name: nginx

    image: nginx


    volumeMounts:

    - name: config-volume

      mountPath: /config


  volumes:

  - name: config-volume

    configMap:

      name: app-config
```

---

# Step 8 - Create Pod

Run:

```bash
kubectl apply -f yaml/lab-13/configmap-volume.yaml
```

---

Enter:

```bash
kubectl exec -it configmap-volume-demo -- bash
```

Check:

```bash
ls /config
```

You will see:

```
APP_NAME
APP_ENV
APP_VERSION
```

Exit:

```bash
exit
```

---

# Verification

Run:

```bash
kubectl get configmap
```

```bash
kubectl get pods
```

```bash
kubectl describe configmap app-config
```

---

# Troubleshooting

## ConfigMap not found

Check:

```bash
kubectl get configmap
```

Make sure name matches:

```yaml
name: app-config
```

---

## Environment variable missing

Check:

```bash
kubectl describe pod configmap-demo
```

---

# Interview Questions

## 1. What is ConfigMap?

ConfigMap stores non-sensitive configuration data.

---

## 2. Why use ConfigMap?

To separate application configuration from application code.

---

## 3. Can ConfigMap store passwords?

No.

Passwords should use Kubernetes Secret.

---

## 4. How can Pods use ConfigMap?

Two ways:

1. Environment variables

2. Mounted files

---

## 5. Is ConfigMap encrypted?

No.

---

# Commands Learned

```bash
kubectl create configmap

kubectl get configmap

kubectl describe configmap

kubectl get configmap -o yaml

kubectl apply -f file.yaml
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod configmap-demo
```

```bash
kubectl delete pod configmap-volume-demo
```

Delete ConfigMap:

```bash
kubectl delete configmap app-config
```

---

# Navigation

Previous:

[Lab 12 - Ingress](lab-12-ingress.md)

Next:

[Lab 14 - Secret](lab-14-secret.md)

---

# Lab Completed ✅

You learned:

- ConfigMap
- Environment variables
- Volume mounting
- Configuration management