# LAB 35 - Kubernetes DNS Troubleshooting

## Objective

In this lab, you will learn:

- What Kubernetes DNS is
- How CoreDNS works
- How Services are resolved
- How to troubleshoot DNS problems
- How Pods communicate using service names

---

# What is Kubernetes DNS?

Kubernetes has an internal DNS system that allows Pods to communicate using names instead of IP addresses.

Example:

Instead of:

```
http://10.96.50.10
```

We use:

```
http://nginx-service
```

Kubernetes DNS converts the name into an IP address.

---

# Kubernetes DNS Architecture

```
Application Pod

      |

DNS Query

      |

CoreDNS

      |

Kubernetes API Server

      |

Service IP

```

---

# What is CoreDNS?

CoreDNS is the DNS server used by Kubernetes.

It runs inside:

```
kube-system namespace
```

Check:

```bash
kubectl get pods -n kube-system
```

Example:

```
coredns-xxxxx   Running
```

---

# Kubernetes DNS Naming

A Service can be accessed using:

```
service-name.namespace.svc.cluster.local
```

Example:

```
nginx-service.default.svc.cluster.local
```

Short form:

```
nginx-service
```

works inside the same namespace.

---

# Common DNS Problems

## 1. CoreDNS Not Running

Check:

```bash
kubectl get pods -n kube-system
```

If CoreDNS is not Running:

DNS will fail.

---

## 2. Wrong Service Name

Example:

Application uses:

```
mysql-service
```

but actual service:

```
mysql-db
```

Result:

```
DNS resolution failed
```

---

## 3. Wrong Namespace

Example:

Service exists in:

```
database namespace
```

Application searches:

```
default namespace
```

Result:

```
service not found
```

---

## 4. Network Problems

Pod cannot communicate with CoreDNS.

---

# Troubleshooting Flow

```
Application Cannot Reach Service

          |

Check DNS

          |

Check CoreDNS

          |

Check Service Name

          |

Check Namespace

          |

Test DNS Resolution

```

---

# Lab Steps

# Step 1 - Create Nginx Deployment

Create:

```bash
touch yaml/lab-35/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: nginx


spec:

  replicas: 1


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

        image: nginx

        ports:

        - containerPort: 80
```

---

# Step 2 - Create Service

Create:

```bash
touch yaml/lab-35/nginx-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: nginx-service


spec:

  selector:

    app: nginx


  ports:

  - port: 80

    targetPort: 80
```

---

# Step 3 - Deploy Application

Run:

```bash
kubectl apply -f yaml/lab-35/nginx-deployment.yaml
```

Run:

```bash
kubectl apply -f yaml/lab-35/nginx-service.yaml
```

---

# Step 4 - Check Service

Run:

```bash
kubectl get svc
```

Example:

```
nginx-service
```

---

# Step 5 - Create DNS Test Pod

Run:

```bash
kubectl run dns-test \
--image=busybox \
-it \
--rm \
-- sh
```

Inside the Pod:

---

Test DNS:

```bash
nslookup nginx-service
```

Expected:

```
Name: nginx-service

Address: 10.xx.xx.xx
```

---

# Step 6 - Test Service Connection

Inside dns-test Pod:

```bash
wget -qO- http://nginx-service
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

# Step 7 - Check CoreDNS

Run:

```bash
kubectl get pods -n kube-system
```

Find:

```
coredns
```

---

Describe CoreDNS:

```bash
kubectl describe pod -n kube-system POD_NAME
```

---

# Step 8 - Check DNS Configuration

Run:

```bash
kubectl get configmap coredns -n kube-system
```

---

View:

```bash
kubectl describe configmap coredns -n kube-system
```

---

# Important Debug Commands

## Check DNS Pods

```bash
kubectl get pods -n kube-system
```

---

## Test DNS Resolution

```bash
nslookup SERVICE_NAME
```

---

## Check Service

```bash
kubectl get svc
```

---

## Check Endpoints

```bash
kubectl get endpoints
```

---

## Check Pod DNS Settings

```bash
kubectl exec POD_NAME -- cat /etc/resolv.conf
```

---

# Verification

Run:

```bash
kubectl get pods
```

Run:

```bash
kubectl get svc
```

Test:

```bash
nslookup nginx-service
```

DNS should return Service IP.

---

# Troubleshooting Checklist

When DNS is not working:

1. Check CoreDNS

```bash
kubectl get pods -n kube-system
```

2. Check Service exists

```bash
kubectl get svc
```

3. Check Service name

4. Check namespace

5. Check endpoints

```bash
kubectl get endpoints
```

6. Test DNS from another Pod

---

# Interview Questions

## 1. What is CoreDNS?

CoreDNS is the DNS server used by Kubernetes clusters.

---

## 2. Where does CoreDNS run?

Inside:

```
kube-system namespace
```

---

## 3. How does Pod communicate with Service?

Using Kubernetes DNS.

Example:

```
service-name.namespace.svc.cluster.local
```

---

## 4. What happens if CoreDNS fails?

Service name resolution stops.

---

## 5. How do you test DNS inside Kubernetes?

Use:

```bash
nslookup
```

from a test Pod.

---

# Commands Learned

```bash
kubectl get pods -n kube-system

kubectl get svc

kubectl get endpoints

kubectl exec

nslookup

cat /etc/resolv.conf
```

---

# Cleanup

Delete Deployment:

```bash
kubectl delete deployment nginx
```

Delete Service:

```bash
kubectl delete svc nginx-service
```

---

# Navigation

Previous:

[Lab 34 - Service Troubleshooting](lab-34-service-troubleshooting.md)

Next:

[Lab 36 - Kubernetes Network Troubleshooting](lab-36-network-troubleshooting.md)

---

# Lab Completed ✅

You learned:

- Kubernetes DNS
- CoreDNS
- Service discovery
- DNS troubleshooting