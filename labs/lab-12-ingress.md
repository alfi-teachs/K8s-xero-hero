# LAB 12 - Kubernetes Ingress

## Objective

In this lab, you will learn:

- What is Kubernetes Ingress
- Why Ingress is required
- Install Ingress Controller
- Create Ingress rules
- Route traffic using URL paths
- Access applications using hostname

---

# What is Ingress?

Ingress is a Kubernetes object that manages external HTTP and HTTPS traffic.

It acts like a reverse proxy.

Example:

Without Ingress:

```
User

 |
 |
Service 1
Service 2
Service 3
```

Each service needs its own external access.

---

With Ingress:

```
             User

              |

          Ingress Controller

              |

       ------------------

       |        |        |

    Service1 Service2 Service3
```

One entry point can route traffic to multiple applications.

---

# Real World Example

A company has:

Frontend:

```
example.com/
```

Backend API:

```
example.com/api
```

Admin:

```
example.com/admin
```

Ingress handles routing.

---

# Ingress Components

## Ingress Resource

Contains routing rules.

Example:

```
/app

/api

/admin
```

---

## Ingress Controller

Actually performs routing.

Examples:

- Nginx Ingress Controller
- Traefik
- AWS ALB Controller

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

# Step 1 - Enable Ingress Controller

For Minikube:

```bash
minikube addons enable ingress
```

Expected:

```
The 'ingress' addon is enabled
```

---

# Step 2 - Verify Ingress Controller

Run:

```bash
kubectl get pods -n ingress-nginx
```

Expected:

```
ingress-nginx-controller Running
```

---

# Step 3 - Create Application Deployment

Create:

```bash
touch yaml/lab-12/nginx-deployment.yaml
```

Open:

```bash
code yaml/lab-12/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-ingress-demo


spec:

  replicas: 2


  selector:
    matchLabels:
      app: nginx


  template:

    metadata:
      labels:
        app: nginx


    spec:

      containers:

      - name: nginx

        image: nginx:latest

        ports:

        - containerPort: 80
```

---

# Step 4 - Create Service

Create:

```bash
touch yaml/lab-12/nginx-service.yaml
```

Open:

```bash
code yaml/lab-12/nginx-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-ingress-service


spec:

  selector:
    app: nginx


  ports:

  - port: 80

    targetPort: 80
```

---

# Step 5 - Deploy Application

Run:

```bash
kubectl apply -f yaml/lab-12/nginx-deployment.yaml
```

Run:

```bash
kubectl apply -f yaml/lab-12/nginx-service.yaml
```

Check:

```bash
kubectl get pods
```

---

# Step 6 - Create Ingress Rule

Create:

```bash
touch yaml/lab-12/nginx-ingress.yaml
```

Open:

```bash
code yaml/lab-12/nginx-ingress.yaml
```

Add:

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: nginx-ingress


spec:

  ingressClassName: nginx


  rules:

  - host: nginx.local


    http:

      paths:

      - path: /

        pathType: Prefix


        backend:

          service:

            name: nginx-ingress-service

            port:

              number: 80
```

---

# Step 7 - Apply Ingress

Run:

```bash
kubectl apply -f yaml/lab-12/nginx-ingress.yaml
```

Expected:

```
ingress.networking.k8s.io/nginx-ingress created
```

---

# Step 8 - Check Ingress

Run:

```bash
kubectl get ingress
```

Example:

```
NAME

nginx-ingress
```

---

# Step 9 - Configure Local DNS

Find Minikube IP:

```bash
minikube ip
```

Example:

```
192.168.49.2
```

Edit hosts file.

Windows:

Open:

```
C:\Windows\System32\drivers\etc\hosts
```

Add:

```
192.168.49.2 nginx.local
```

Save.

---

# Step 10 - Access Application

Open browser:

```
http://nginx.local
```

Expected:

```
Welcome to nginx!
```

---

# Traffic Flow

```
Browser

 |

nginx.local

 |

Ingress Controller

 |

Ingress Rule

 |

Service

 |

Pod

 |

Nginx Container
```

---

# Verification

Run:

```bash
kubectl get ingress
```

```bash
kubectl describe ingress nginx-ingress
```

```bash
kubectl get svc
```

```bash
kubectl get pods
```

---

# Troubleshooting

## Ingress has no address

Check controller:

```bash
kubectl get pods -n ingress-nginx
```

---

## Website not opening

Check hosts file:

```
nginx.local
```

should point to:

```
Minikube IP
```

---

## Ingress not routing

Check service:

```bash
kubectl get endpoints
```

---

# Interview Questions

## 1. What is Ingress?

Ingress manages external HTTP and HTTPS traffic to Kubernetes services.

---

## 2. What is Ingress Controller?

A component that implements Ingress rules.

---

## 3. Difference between Service and Ingress?

Service:

Provides network access to Pods.

Ingress:

Routes external HTTP traffic to Services.

---

## 4. Why use Ingress?

To expose multiple applications using one external endpoint.

---

## 5. What is a reverse proxy?

A server that receives client requests and forwards them to backend services.

---

# Commands Learned

```bash
minikube addons enable ingress

kubectl get ingress

kubectl describe ingress

kubectl get endpoints

kubectl apply -f file.yaml
```

---

# Cleanup

Delete Ingress:

```bash
kubectl delete ingress nginx-ingress
```

Delete Service:

```bash
kubectl delete service nginx-ingress-service
```

Delete Deployment:

```bash
kubectl delete deployment nginx-ingress-demo
```

Disable addon:

```bash
minikube addons disable ingress
```

---

# Navigation

Previous:

[Lab 11 - LoadBalancer Service](lab-11-loadbalancer.md)

Next:

[Lab 13 - ConfigMap](lab-13-configmap.md)

---

# Lab Completed ✅

You learned:

- Ingress Controller
- Ingress Resource
- URL routing
- Reverse proxy concept
- Production style traffic management