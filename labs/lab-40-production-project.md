# LAB 40 - Production Kubernetes Project

## Objective

In this final project, you will build a production-style Kubernetes application.

You will learn:

- Production Kubernetes architecture
- Application deployment
- Configuration management
- Secret management
- Ingress routing
- Autoscaling
- Health monitoring
- Resource management
- Kubernetes best practices

---

# Production Architecture

```
                 User

                  |

              Ingress

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

            Database Pods


                  |

             Persistent Storage

```

---

# Project Components

## Namespace

Separate application resources.

Example:

```
production
```

---

## Frontend

Responsible for:

- User interface
- Web pages

Technology:

```
Nginx
```

---

## Backend

Responsible for:

- API
- Business logic

---

## Database

Responsible for:

- Data storage

Example:

```
MySQL
```

---

## Ingress

Provides external access.

Example:

```
app.example.com
```

---

## HPA

Automatically scales Pods.

Example:

```
CPU > 70%

Increase replicas
```

---

# Production Folder Structure

```
yaml/lab-40/

├── namespace.yaml

├── configmap.yaml

├── secret.yaml

├── frontend/

│   ├── deployment.yaml

│   └── service.yaml


├── backend/

│   ├── deployment.yaml

│   └── service.yaml


├── database/

│   ├── deployment.yaml

│   └── service.yaml


├── ingress.yaml

├── hpa.yaml

└── pvc.yaml

```

---

# Lab Steps

# Step 1 - Create Namespace

File:

```
yaml/lab-40/namespace.yaml
```

Content:

```yaml
apiVersion: v1

kind: Namespace


metadata:

  name: production
```

Apply:

```bash
kubectl apply -f yaml/lab-40/namespace.yaml
```

---

# Step 2 - Create ConfigMap

File:

```
yaml/lab-40/configmap.yaml
```

Content:

```yaml
apiVersion: v1

kind: ConfigMap


metadata:

  name: app-config

  namespace: production


data:

  APP_ENV: production

  APP_NAME: kubernetes-demo
```

Apply:

```bash
kubectl apply -f yaml/lab-40/configmap.yaml
```

---

# Step 3 - Create Secret

File:

```
yaml/lab-40/secret.yaml
```

Content:

```yaml
apiVersion: v1

kind: Secret


metadata:

  name: database-secret

  namespace: production


type: Opaque


data:

  PASSWORD: YWRtaW4=
```

Apply:

```bash
kubectl apply -f yaml/lab-40/secret.yaml
```

---

# Step 4 - Create Frontend Deployment

Create folder:

```bash
mkdir yaml/lab-40/frontend
```

File:

```
yaml/lab-40/frontend/deployment.yaml
```

Content:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: frontend

  namespace: production


spec:

  replicas: 3


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


        resources:

          requests:

            cpu: "100m"

            memory: "128Mi"


          limits:

            cpu: "500m"

            memory: "256Mi"
```

---

Apply:

```bash
kubectl apply -f yaml/lab-40/frontend/deployment.yaml
```

---

# Step 5 - Create Frontend Service

File:

```
yaml/lab-40/frontend/service.yaml
```

Content:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: frontend-service

  namespace: production


spec:

  selector:

    app: frontend


  ports:

  - port: 80

    targetPort: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-40/frontend/service.yaml
```

---

# Step 6 - Create Backend Deployment

File:

```
yaml/lab-40/backend/deployment.yaml
```

Content:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: backend

  namespace: production


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
kubectl apply -f yaml/lab-40/backend/deployment.yaml
```

---

# Step 7 - Create Backend Service

File:

```
yaml/lab-40/backend/service.yaml
```

Content:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: backend-service

  namespace: production


spec:

  selector:

    app: backend


  ports:

  - port: 80

    targetPort: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-40/backend/service.yaml
```

---

# Step 8 - Create Horizontal Pod Autoscaler

File:

```
yaml/lab-40/hpa.yaml
```

Content:

```yaml
apiVersion: autoscaling/v2

kind: HorizontalPodAutoscaler


metadata:

  name: frontend-hpa

  namespace: production


spec:

  scaleTargetRef:

    apiVersion: apps/v1

    kind: Deployment

    name: frontend


  minReplicas: 3

  maxReplicas: 10


  metrics:

  - type: Resource

    resource:

      name: cpu

      target:

        type: Utilization

        averageUtilization: 70
```

Apply:

```bash
kubectl apply -f yaml/lab-40/hpa.yaml
```

---

# Step 9 - Create Ingress

File:

```
yaml/lab-40/ingress.yaml
```

Content:

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress


metadata:

  name: app-ingress

  namespace: production


spec:

  rules:

  - host: app.local

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: frontend-service

            port:

              number: 80
```

Apply:

```bash
kubectl apply -f yaml/lab-40/ingress.yaml
```

---

# Step 10 - Verify Production Environment

Check all resources:

```bash
kubectl get all -n production
```

Expected:

```
Frontend Pods

Backend Pods

Services

Deployments
```

---

# Check HPA

```bash
kubectl get hpa -n production
```

---

# Check Ingress

```bash
kubectl get ingress -n production
```

---

# Check Configuration

```bash
kubectl get configmap -n production
```

---

# Check Secrets

```bash
kubectl get secret -n production
```

---

# Production Troubleshooting

## Check Pods

```bash
kubectl get pods -n production
```

---

## View Logs

```bash
kubectl logs POD_NAME -n production
```

---

## Describe Resources

```bash
kubectl describe pod POD_NAME -n production
```

---

## Check Events

```bash
kubectl get events -n production
```

---

# Production Best Practices Learned

✅ Use namespaces

✅ Use resource limits

✅ Use ConfigMaps

✅ Use Secrets

✅ Use Services

✅ Use Ingress

✅ Use Autoscaling

✅ Monitor applications

✅ Troubleshoot failures

---

# Interview Questions

## 1. What makes Kubernetes production ready?

Features:

- Scaling
- Self healing
- Networking
- Security
- Monitoring

---

## 2. Why use namespaces?

To separate environments and resources.

---

## 3. What is HPA?

Horizontal Pod Autoscaler automatically changes Pod count based on metrics.

---

## 4. Why use Ingress?

To expose applications externally using HTTP/HTTPS routing.

---

## 5. Difference between ConfigMap and Secret?

ConfigMap:

Stores normal configuration.

Secret:

Stores sensitive data.

---

# Commands Learned

```bash
kubectl apply

kubectl get all

kubectl get ingress

kubectl get hpa

kubectl logs

kubectl describe
```

---

# Cleanup

Delete complete project:

```bash
kubectl delete namespace production
```

---

# Navigation

Previous:

[Lab 39 - Three Tier Application](lab-39-three-tier-app.md)

---

# Course Completed 🎉

You completed:

## Kubernetes Xero to Hero

40 Hands-on Kubernetes Labs

From:

Beginner Kubernetes

to

Production Kubernetes Engineer