# LAB 10 - Kubernetes NodePort Service

## Objective

In this lab, you will learn:

- What is NodePort Service
- Difference between ClusterIP and NodePort
- Expose application outside Kubernetes cluster
- Access application using browser
- Understand NodePort traffic flow

---

# What is NodePort?

NodePort exposes a Kubernetes Service outside the cluster.

It opens a port on every Kubernetes node.

Example:

```
Browser

   |

Node IP:30080

   |

NodePort Service

   |

Pod
```

---

# ClusterIP vs NodePort

## ClusterIP

Used for internal communication.

Example:

```
Backend --> Database
```

Accessible only inside cluster.

---

## NodePort

Used for external access.

Example:

```
Laptop Browser --> Kubernetes Node --> Pod
```

---

# NodePort Port Range

Kubernetes NodePort uses:

```
30000 - 32767
```

Example:

```
30080
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

# Step 1 - Create Deployment

Create file:

```bash
touch yaml/lab-10/nginx-deployment.yaml
```

Open:

```bash
code yaml/lab-10/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-nodeport-demo


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
kubectl apply -f yaml/lab-10/nginx-deployment.yaml
```

Check:

```bash
kubectl get pods
```

Expected:

```
3 nginx Pods Running
```

---

# Step 3 - Create NodePort Service

Create:

```bash
touch yaml/lab-10/nginx-nodeport.yaml
```

Open:

```bash
code yaml/lab-10/nginx-nodeport.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-nodeport-service


spec:

  type: NodePort


  selector:
    app: nginx


  ports:

  - port: 80

    targetPort: 80

    nodePort: 30080
```

---

# Understanding YAML

## Service Type

```yaml
type: NodePort
```

Creates external access.

---

## Port

```yaml
port: 80
```

Service port.

---

## Target Port

```yaml
targetPort: 80
```

Container port.

---

## Node Port

```yaml
nodePort: 30080
```

Port exposed on the node.

---

# Step 4 - Create Service

Run:

```bash
kubectl apply -f yaml/lab-10/nginx-nodeport.yaml
```

Expected:

```
service/nginx-nodeport-service created
```

---

# Step 5 - Verify Service

Run:

```bash
kubectl get svc
```

Expected:

```
NAME                     TYPE

nginx-nodeport-service   NodePort
```

Example:

```
80:30080/TCP
```

Meaning:

```
Service Port: 80

Node Port: 30080
```

---

# Step 6 - Get Minikube IP

Run:

```bash
minikube ip
```

Example:

```
192.168.49.2
```

---

# Step 7 - Access Application

Open browser:

```
http://MINIKUBE-IP:30080
```

Example:

```
http://192.168.49.2:30080
```

You should see:

```
Welcome to nginx!
```

---

# Step 8 - Alternative Method

Minikube provides a shortcut:

```bash
minikube service nginx-nodeport-service
```

This automatically opens the browser.

---

# Step 9 - Check Service Endpoints

Run:

```bash
kubectl get endpoints
```

Example:

```
nginx-nodeport-service

10.244.0.5
10.244.0.6
10.244.0.7
```

---

# Traffic Flow

Complete request flow:

```
User Browser

      |

192.168.49.2:30080

      |

NodePort Service

      |

ClusterIP

      |

Pod Port 80

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

## Browser not opening

Check Minikube:

```bash
minikube status
```

---

## Service not routing traffic

Check labels:

```bash
kubectl get pods --show-labels
```

Service selector:

```yaml
selector:
  app: nginx
```

Pod labels must match.

---

## Port not accessible

Check:

```bash
kubectl describe service nginx-nodeport-service
```

---

# Interview Questions

## 1. What is NodePort?

NodePort exposes Kubernetes Service outside the cluster.

---

## 2. What port range does NodePort use?

```
30000-32767
```

---

## 3. Difference between ClusterIP and NodePort?

ClusterIP:

Internal communication.

NodePort:

External access.

---

## 4. Does NodePort create a new Pod?

No.

It only exposes existing Pods.

---

## 5. How does NodePort find Pods?

Using labels and selectors.

---

# Commands Learned

```bash
kubectl get svc

kubectl describe service

kubectl get endpoints

minikube ip

minikube service service-name
```

---

# Cleanup

Delete Service:

```bash
kubectl delete service nginx-nodeport-service
```

Delete Deployment:

```bash
kubectl delete deployment nginx-nodeport-demo
```

---

# Navigation

Previous:

[Lab 09 - ClusterIP Service](lab-09-clusterip-service.md)

Next:

[Lab 11 - LoadBalancer Service](lab-11-loadbalancer.md)

---

# Lab Completed ✅

You learned:

- NodePort Service
- External access
- Port mapping
- Browser access
- Service networking