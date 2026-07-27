# LAB 36 - Kubernetes Network Troubleshooting

## Objective

In this lab, you will learn:

- How Kubernetes networking works
- Pod-to-Pod communication
- Service networking
- Network troubleshooting commands
- How to debug connection problems

---

# What is Kubernetes Networking?

Kubernetes provides communication between:

- Pods
- Services
- Nodes
- External users

Every Pod receives its own IP address.

Example:

```
Pod A
10.244.1.5

      |

Network

      |

Pod B
10.244.2.8
```

Pods can communicate directly with each other.

---

# Kubernetes Network Rules

Kubernetes networking follows:

## Rule 1

Every Pod gets a unique IP.

Example:

```
Pod IP:

10.244.1.10
```

---

## Rule 2

Pods can communicate without NAT.

Example:

```
Frontend Pod

   |

Backend Pod

```

---

## Rule 3

Services provide stable access.

Example:

```
Pod IP changes

but

Service IP remains same
```

---

# Kubernetes Networking Components

## 1. CNI Plugin

CNI manages Pod networking.

Examples:

- Calico
- Flannel
- Cilium

---

## 2. kube-proxy

Handles Service networking.

It creates rules for:

- ClusterIP
- NodePort
- LoadBalancer

---

## 3. CoreDNS

Provides service discovery.

Example:

```
backend-service.default.svc.cluster.local
```

---

# Common Network Problems

## 1. Pod Cannot Reach Another Pod

Possible reasons:

- Network policy blocking traffic
- Application not listening
- Wrong IP address

---

## 2. Service Cannot Reach Pod

Possible reasons:

- Wrong selector
- Wrong port
- No endpoints

---

## 3. DNS Resolution Failure

Possible reasons:

- CoreDNS issue
- Wrong service name

---

## 4. Port Problem

Example:

Application listens on:

```
8080
```

Service sends traffic to:

```
80
```

Connection fails.

---

# Troubleshooting Flow

```
Connection Failed

       |

Check Pod Status

       |

Check IP Address

       |

Check Service

       |

Check Endpoints

       |

Test Connectivity

       |

Fix Issue

```

---

# Lab Steps

# Step 1 - Create Backend Application

Create:

```bash
touch yaml/lab-36/backend.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: backend-pod

  labels:

    app: backend


spec:

  containers:

  - name: backend

    image: nginx

    ports:

    - containerPort: 80
```

---

# Step 2 - Create Frontend Test Pod

Create:

```bash
touch yaml/lab-36/frontend.yaml
```

Add:

```yaml
apiVersion: v1

kind: Pod


metadata:

  name: frontend-pod


spec:

  containers:

  - name: frontend

    image: busybox

    command:

    - sleep

    - "3600"
```

---

# Step 3 - Create Backend Service

Create:

```bash
touch yaml/lab-36/backend-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: backend-service


spec:

  selector:

    app: backend


  ports:

  - port: 80

    targetPort: 80
```

---

# Step 4 - Deploy Resources

Run:

```bash
kubectl apply -f yaml/lab-36/
```

---

Check:

```bash
kubectl get pods
```

Expected:

```
backend-pod     Running
frontend-pod    Running
```

---

# Step 5 - Test Pod Connectivity

Enter frontend pod:

```bash
kubectl exec -it frontend-pod -- sh
```

Test backend service:

```bash
wget -qO- http://backend-service
```

Expected:

```
Welcome to nginx!
```

Exit:

```bash
exit
```

---

# Step 6 - Check Pod IP Addresses

Run:

```bash
kubectl get pods -o wide
```

Example:

```
NAME            IP

backend-pod     10.244.1.5

frontend-pod    10.244.1.6
```

---

# Step 7 - Check Service Endpoints

Run:

```bash
kubectl get endpoints backend-service
```

Expected:

```
10.244.x.x:80
```

---

# Step 8 - Debug DNS

Run:

```bash
kubectl exec frontend-pod -- nslookup backend-service
```

Expected:

```
Name: backend-service
Address: 10.xx.xx.xx
```

---

# Important Debug Commands

## Check Pods

```bash
kubectl get pods
```

---

## Check Pod IP

```bash
kubectl get pods -o wide
```

---

## Check Services

```bash
kubectl get svc
```

---

## Check Endpoints

```bash
kubectl get endpoints
```

---

## Test Connection

```bash
kubectl exec POD_NAME -- curl SERVICE_NAME
```

---

## Check Network Events

```bash
kubectl get events
```

---

# Verification

Run:

```bash
kubectl get pods
```

Run:

```bash
kubectl get endpoints
```

Test:

```bash
kubectl exec frontend-pod -- wget -qO- http://backend-service
```

You should receive:

```
Welcome to nginx!
```

---

# Troubleshooting Checklist

When networking fails:

## 1. Check Pod Status

```bash
kubectl get pods
```

---

## 2. Check Pod IP

```bash
kubectl get pods -o wide
```

---

## 3. Check Service Selector

```bash
kubectl describe svc SERVICE_NAME
```

---

## 4. Check Endpoints

```bash
kubectl get endpoints
```

---

## 5. Test DNS

```bash
nslookup SERVICE_NAME
```

---

## 6. Check Ports

Verify:

```
containerPort

targetPort

service port
```

---

# Interview Questions

## 1. How do Pods communicate in Kubernetes?

Pods communicate using Pod IP addresses through the Kubernetes network.

---

## 2. What provides networking for Pods?

CNI plugin.

Examples:

```
Calico
Flannel
Cilium
```

---

## 3. What is kube-proxy?

A component that manages Service networking.

---

## 4. Why use Services instead of Pod IP?

Because Pod IP changes when Pods restart.

---

## 5. How do you troubleshoot network issues?

Commands:

```bash
kubectl get pods -o wide

kubectl describe svc

kubectl get endpoints

kubectl exec
```

---

# Commands Learned

```bash
kubectl get pods -o wide

kubectl get endpoints

kubectl exec

kubectl describe svc

kubectl get events
```

---

# Cleanup

Delete Pods:

```bash
kubectl delete pod backend-pod frontend-pod
```

Delete Service:

```bash
kubectl delete svc backend-service
```

---

# Navigation

Previous:

[Lab 35 - DNS Troubleshooting](lab-35-dns-troubleshooting.md)

Next:

[Lab 37 - Prometheus Monitoring](lab-37-prometheus.md)

---

# Lab Completed ✅

You learned:

- Kubernetes networking
- Pod communication
- Service connectivity
- Network troubleshooting