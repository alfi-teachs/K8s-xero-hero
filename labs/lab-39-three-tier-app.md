# LAB 39 - Three Tier Application on Kubernetes

## Objective

In this lab, you will learn:

- What three-tier architecture is
- Deploy frontend application
- Deploy backend application
- Deploy database
- Connect applications using Services
- Use ConfigMaps
- Use Secrets
- Deploy a real application architecture

---

# What is Three Tier Architecture?

Three-tier architecture separates an application into three layers.

## 1. Frontend Layer

Responsible for user interface.

Example:

```
Website
HTML
CSS
JavaScript
```

Technology:

```
Nginx
React
Angular
```

---

## 2. Backend Layer

Responsible for business logic.

Example:

```
API
Authentication
Application Logic
```

Technology:

```
Java
Python
Node.js
```

---

## 3. Database Layer

Stores application data.

Example:

```
MySQL
PostgreSQL
MongoDB
```

---

# Kubernetes Architecture

Our deployment:

```
                 User

                  |

            Frontend Service

                  |

            Frontend Pods


                  |

            Backend Service

                  |

            Backend Pods


                  |

            Database Service

                  |

            Database Pod

```

---

# Project Structure

```
yaml/lab-39/

├── namespace.yaml

├── frontend-deployment.yaml

├── frontend-service.yaml

├── backend-deployment.yaml

├── backend-service.yaml

├── mysql-deployment.yaml

├── mysql-service.yaml

├── mysql-secret.yaml

└── configmap.yaml

```

---

# Lab Steps

# Step 1 - Create Namespace

Create:

```
yaml/lab-39/namespace.yaml
```

Content:

```yaml
apiVersion: v1

kind: Namespace


metadata:

  name: three-tier
```

Apply:

```bash
kubectl apply -f yaml/lab-39/namespace.yaml
```

---

# Step 2 - Create Database Secret

Create:

```
yaml/lab-39/mysql-secret.yaml
```

Content:

```yaml
apiVersion: v1

kind: Secret


metadata:

  name: mysql-secret

  namespace: three-tier


type: Opaque


data:

  MYSQL_ROOT_PASSWORD: YWRtaW4=
```

Apply:

```bash
kubectl apply -f yaml/lab-39/mysql-secret.yaml
```

---

# Step 3 - Deploy MySQL Database

Create:

```
yaml/lab-39/mysql-deployment.yaml
```

Content:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: mysql

  namespace: three-tier


spec:

  replicas: 1


  selector:

    matchLabels:

      app: mysql


  template:

    metadata:

      labels:

        app: mysql


    spec:

      containers:

      - name: mysql

        image: mysql:8


        env:

        - name: MYSQL_ROOT_PASSWORD

          valueFrom:

            secretKeyRef:

              name: mysql-secret

              key: MYSQL_ROOT_PASSWORD


        ports:

        - containerPort: 3306
```

Apply:

```bash
kubectl apply -f yaml/lab-39/mysql-deployment.yaml
```

---

# Step 4 - Create Database Service

Create:

```
yaml/lab-39/mysql-service.yaml
```

Content:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: mysql

  namespace: three-tier


spec:

  selector:

    app: mysql


  ports:

  - port: 3306

    targetPort: 3306
```

Apply:

```bash
kubectl apply -f yaml/lab-39/mysql-service.yaml
```

---

# Step 5 - Deploy Backend

Create:

```
yaml/lab-39/backend-deployment.yaml
```

Content:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: backend

  namespace: three-tier


spec:

  replicas: 2


  selector:

    matchLabels:

      app: backend


  template:

    metadata:

      labels:

        app: backend


    spec:

      containers:

      - name: backend

        image: nginx

        ports:

        - containerPort: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-39/backend-deployment.yaml
```

---

# Step 6 - Backend Service

Create:

```
yaml/lab-39/backend-service.yaml
```

Content:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: backend

  namespace: three-tier


spec:

  selector:

    app: backend


  ports:

  - port: 80

    targetPort: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-39/backend-service.yaml
```

---

# Step 7 - Deploy Frontend

Create:

```
yaml/lab-39/frontend-deployment.yaml
```

Content:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: frontend

  namespace: three-tier


spec:

  replicas: 2


  selector:

    matchLabels:

      app: frontend


  template:

    metadata:

      labels:

        app: frontend


    spec:

      containers:

      - name: frontend

        image: nginx

        ports:

        - containerPort: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-39/frontend-deployment.yaml
```

---

# Step 8 - Frontend Service

Create:

```
yaml/lab-39/frontend-service.yaml
```

Content:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: frontend

  namespace: three-tier


spec:

  type: NodePort


  selector:

    app: frontend


  ports:

  - port: 80

    targetPort: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-39/frontend-service.yaml
```

---

# Step 9 - Verify Application

Check namespace:

```bash
kubectl get all -n three-tier
```

Expected:

```
frontend pods

backend pods

mysql pod

services
```

---

# Access Frontend

Run:

```bash
minikube service frontend -n three-tier --url
```

Open URL.

Expected:

```
Welcome to nginx
```

---

# Troubleshooting Commands

Check Pods:

```bash
kubectl get pods -n three-tier
```

Check Services:

```bash
kubectl get svc -n three-tier
```

Check Logs:

```bash
kubectl logs POD_NAME -n three-tier
```

Describe:

```bash
kubectl describe pod POD_NAME -n three-tier
```

---

# Interview Questions

## 1. What is three-tier architecture?

Architecture where frontend, backend, and database are separated.

---

## 2. Why separate application layers?

Benefits:

- Easy scaling
- Better security
- Easier maintenance

---

## 3. How does frontend communicate with backend?

Using Kubernetes Service.

---

## 4. How does backend connect to database?

Using database Service name.

Example:

```
mysql.three-tier.svc.cluster.local
```

---

## 5. Why use Secrets?

To store sensitive information like passwords.

---

# Commands Learned

```bash
kubectl apply

kubectl get all

kubectl get pods

kubectl get svc

kubectl logs

kubectl describe
```

---

# Cleanup

Delete project:

```bash
kubectl delete namespace three-tier
```

---

# Navigation

Previous:

[Lab 38 - Grafana Monitoring](lab-38-grafana.md)

Next:

[Lab 40 - Production Kubernetes Project](lab-40-production-project.md)

---

# Lab Completed ✅

You built:

- Frontend layer
- Backend layer
- Database layer
- Kubernetes multi-tier application