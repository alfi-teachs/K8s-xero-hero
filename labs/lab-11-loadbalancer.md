# LAB 11 - Kubernetes LoadBalancer Service

## Objective

In this lab, you will learn:

- What is LoadBalancer Service
- Difference between NodePort and LoadBalancer
- How cloud providers expose applications
- Use Minikube Tunnel
- Access application using external IP

---

# What is LoadBalancer?

LoadBalancer exposes Kubernetes applications to external users.

In cloud platforms:

AWS:

```
Internet
   |
AWS Load Balancer
   |
Kubernetes Service
   |
Pods
```

Azure:

```
Internet
   |
Azure Load Balancer
   |
Kubernetes Service
   |
Pods
```

---

# Why Do We Need LoadBalancer?

NodePort:

```
User

NodeIP:30080

Service

Pods
```

Problem:

- Need to remember port
- Not production friendly

LoadBalancer:

```
User

External IP

Service

Pods
```

Provides:

- Public IP address
- External access
- Cloud integration

---

# Service Types Comparison

| Service | Access |
|---|---|
| ClusterIP | Inside cluster only |
| NodePort | Outside using Node IP + Port |
| LoadBalancer | External IP |

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

# Step 1 - Create Deployment

Create:

```bash
touch yaml/lab-11/nginx-deployment.yaml
```

Open:

```bash
code yaml/lab-11/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-loadbalancer-demo


spec:

  replicas: 3


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

# Step 2 - Deploy Application

Run:

```bash
kubectl apply -f yaml/lab-11/nginx-deployment.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
3 Pods Running
```

---

# Step 3 - Create LoadBalancer Service

Create:

```bash
touch yaml/lab-11/nginx-loadbalancer.yaml
```

Open:

```bash
code yaml/lab-11/nginx-loadbalancer.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-loadbalancer-service


spec:

  type: LoadBalancer


  selector:
    app: nginx


  ports:

  - port: 80

    targetPort: 80
```

---

# Step 4 - Apply Service

Run:

```bash
kubectl apply -f yaml/lab-11/nginx-loadbalancer.yaml
```

Expected:

```
service/nginx-loadbalancer-service created
```

---

# Step 5 - Check Service

Run:

```bash
kubectl get svc
```

Example:

```
NAME                         TYPE

nginx-loadbalancer-service   LoadBalancer
```

You may see:

```
EXTERNAL-IP

pending
```

Why?

Because Minikube does not automatically provide a cloud load balancer.

---

# Step 6 - Start Minikube Tunnel

Open a new terminal.

Run:

```bash
minikube tunnel
```

Keep this terminal running.

Example:

```
Status:
machine: minikube
state: Running
```

---

# Step 7 - Check External IP Again

In another terminal:

```bash
kubectl get svc
```

Now you should see:

Example:

```
NAME                         EXTERNAL-IP

nginx-loadbalancer-service   127.0.0.1
```

---

# Step 8 - Access Application

Open browser:

```
http://127.0.0.1
```

You should see:

```
Welcome to nginx!
```

---

# Step 9 - Check Service Details

Run:

```bash
kubectl describe service nginx-loadbalancer-service
```

Observe:

- Type
- Cluster IP
- External IP
- Endpoints

---

# Traffic Flow

```
User Browser

      |

External IP

      |

LoadBalancer Service

      |

ClusterIP

      |

Pods

      |

Nginx Container
```

---

# Verification

Run:

```bash
kubectl get deployment

kubectl get pods

kubectl get svc

kubectl get endpoints
```

---

# Troubleshooting

## External IP showing pending

Check tunnel:

```bash
minikube tunnel
```

---

## Application not opening

Check Pods:

```bash
kubectl get pods
```

Check endpoints:

```bash
kubectl get endpoints
```

---

## Service has no endpoints

Check labels:

```bash
kubectl get pods --show-labels
```

Service selector:

```yaml
selector:
  app: nginx
```

must match Pod labels.

---

# Interview Questions

## 1. What is LoadBalancer Service?

A Kubernetes Service that provides external access using a cloud load balancer.

---

## 2. Difference between NodePort and LoadBalancer?

NodePort:

Uses Node IP and port.

LoadBalancer:

Provides external IP.

---

## 3. Why does Minikube need tunnel?

Because Minikube does not have a cloud provider load balancer.

---

## 4. Which cloud services provide LoadBalancer?

Examples:

AWS ELB

Azure Load Balancer

Google Cloud Load Balancer

---

# Commands Learned

```bash
kubectl get svc

kubectl describe svc service-name

minikube tunnel

kubectl get endpoints
```

---

# Cleanup

Delete Service:

```bash
kubectl delete svc nginx-loadbalancer-service
```

Delete Deployment:

```bash
kubectl delete deployment nginx-loadbalancer-demo
```

Stop tunnel:

Press:

```
CTRL + C
```

---

# Navigation

Previous:

[Lab 10 - NodePort Service](lab-10-nodeport-service.md)

Next:

[Lab 12 - Ingress](lab-12-ingress.md)

---

# Lab Completed ✅

You learned:

- LoadBalancer Service
- External IP
- Minikube Tunnel
- Cloud Load Balancer concept