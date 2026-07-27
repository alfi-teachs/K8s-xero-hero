# LAB 34 - Kubernetes Service Troubleshooting

## Objective

In this lab, you will learn:

- How Kubernetes Services work
- Why Services cannot reach Pods
- Troubleshoot Service selector issues
- Check Endpoints
- Debug application connectivity problems

---

# What is a Kubernetes Service?

A Service provides network access to Pods.

Pods are temporary.

Their IP addresses can change.

Service provides a stable address.

Example:

```
User

 |

Service

 |

Pod 1
Pod 2
Pod 3

```

---

# Service Components

A Service contains:

## 1. Service Name

Example:

```
nginx-service
```

---

## 2. Selector

Selector finds Pods.

Example:

Service:

```yaml
selector:
  app: nginx
```

Pod:

```yaml
labels:
  app: nginx
```

They must match.

---

## 3. Port

Service port:

```
80
```

Container port:

```
80
```

Traffic flows:

```
Service Port

      |

Target Port

      |

Container Port

```

---

# Common Service Problems

## 1. Selector Does Not Match Labels

Example:

Service:

```yaml
selector:
  app: web
```

Pod:

```yaml
labels:
  app: nginx
```

Result:

```
No endpoints
```

---

## 2. Wrong Port

Example:

Service:

```
port: 80
```

Pod:

```
containerPort: 8080
```

Result:

```
Connection refused
```

---

## 3. Pods Are Not Running

Check:

```bash
kubectl get pods
```

---

## 4. Service Has No Endpoints

Check:

```bash
kubectl get endpoints
```

---

# Troubleshooting Flow

```
Application Cannot Connect

        |

Check Service

        |

Check Selector

        |

Check Endpoints

        |

Check Pod Labels

        |

Fix Configuration

```

---

# Lab Steps

# Step 1 - Create Deployment With Label

Create:

```bash
touch yaml/lab-34/nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: nginx-deployment


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

        image: nginx

        ports:

        - containerPort: 80
```

---

# Step 2 - Create Deployment

Run:

```bash
kubectl apply -f yaml/lab-34/nginx-deployment.yaml
```

---

Check:

```bash
kubectl get pods --show-labels
```

Expected:

```
app=nginx
```

---

# Step 3 - Create Broken Service

Create:

```bash
touch yaml/lab-34/broken-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: nginx-service


spec:

  selector:

    app: wrong-label


  ports:

  - port: 80

    targetPort: 80
```

---

# Step 4 - Create Service

Run:

```bash
kubectl apply -f yaml/lab-34/broken-service.yaml
```

---

Check:

```bash
kubectl get svc
```

---

# Step 5 - Check Endpoints

Run:

```bash
kubectl get endpoints nginx-service
```

Expected:

```
NAME

nginx-service

ENDPOINTS

<none>
```

Problem:

Service cannot find Pods.

---

# Step 6 - Describe Service

Run:

```bash
kubectl describe service nginx-service
```

Look for:

```
Selector:
app=wrong-label
```

---

# Step 7 - Fix Selector

Edit:

```bash
code yaml/lab-34/broken-service.yaml
```

Change:

```yaml
app: wrong-label
```

to:

```yaml
app: nginx
```

---

Delete old Service:

```bash
kubectl delete service nginx-service
```

Create again:

```bash
kubectl apply -f yaml/lab-34/broken-service.yaml
```

---

# Step 8 - Verify Endpoints

Run:

```bash
kubectl get endpoints nginx-service
```

Expected:

```
10.x.x.x:80
10.x.x.x:80
```

---

# Step 9 - Test Service

Run:

```bash
kubectl run curl-test \
--image=curlimages/curl \
-it \
--rm \
-- curl http://nginx-service
```

Expected:

```
Welcome to nginx!
```

---

# Important Debug Commands

## Check Services

```bash
kubectl get svc
```

---

## Check Service Details

```bash
kubectl describe svc SERVICE_NAME
```

---

## Check Endpoints

```bash
kubectl get endpoints
```

---

## Check Pod Labels

```bash
kubectl get pods --show-labels
```

---

## Test DNS

```bash
nslookup SERVICE_NAME
```

---

# Verification

Run:

```bash
kubectl get svc
```

Run:

```bash
kubectl get endpoints nginx-service
```

Service should show Pod IP addresses.

---

# Troubleshooting Checklist

When Service is not working:

1. Check Pods

```bash
kubectl get pods
```

2. Check Labels

```bash
kubectl get pods --show-labels
```

3. Check Selector

```bash
kubectl describe svc SERVICE_NAME
```

4. Check Endpoints

```bash
kubectl get endpoints
```

5. Check Ports

Verify:

```
service port

targetPort

containerPort
```

---

# Interview Questions

## 1. What is a Kubernetes Service?

A Service provides stable networking to Pods.

---

## 2. How does Service find Pods?

Using labels and selectors.

---

## 3. Why do Services have no endpoints?

Because:

- Selector mismatch
- Pods not running
- Wrong labels

---

## 4. Difference between Pod IP and Service IP?

Pod IP:

Temporary.

Service IP:

Stable.

---

## 5. Which command checks Service endpoints?

```bash
kubectl get endpoints
```

---

# Commands Learned

```bash
kubectl get svc

kubectl describe svc

kubectl get endpoints

kubectl get pods --show-labels

kubectl run
```

---

# Cleanup

Delete Service:

```bash
kubectl delete svc nginx-service
```

Delete Deployment:

```bash
kubectl delete deployment nginx-deployment
```

---

# Navigation

Previous:

[Lab 33 - Pending Pods Troubleshooting](lab-33-pending-pods.md)

Next:

[Lab 35 - DNS Troubleshooting](lab-35-dns-troubleshooting.md)

---

# Lab Completed ✅

You learned:

- Service networking
- Labels and selectors
- Endpoint troubleshooting
- Kubernetes connectivity debugging