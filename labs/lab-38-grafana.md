# LAB 38 - Grafana Monitoring

## Objective

In this lab, you will learn:

- What Grafana is
- Difference between Prometheus and Grafana
- Connect Grafana with Prometheus
- Create dashboards
- Visualize Kubernetes metrics

---

# What is Grafana?

Grafana is a visualization and dashboard tool.

It displays data using:

- Graphs
- Charts
- Tables
- Alerts

Example:

Prometheus stores:

```
CPU = 70%
Memory = 500MB
Pods = 10
```

Grafana displays:

```
CPU Usage Graph

70%
|
|
|
+----------------
 Time
```

---

# Prometheus vs Grafana

| Prometheus | Grafana |
|---|---|
| Collects metrics | Displays metrics |
| Stores time-series data | Creates dashboards |
| Query engine | Visualization tool |

---

# Monitoring Architecture

```
Kubernetes Cluster

        |

 Metrics

        |

 Prometheus

        |

 Data Source

        |

 Grafana

        |

 Dashboard

```

---

# Real World Example

Production Kubernetes cluster:

Prometheus collects:

```
Node CPU
Pod Memory
Container Restart Count
```

Grafana shows:

```
Kubernetes Dashboard

CPU Graph
Memory Graph
Pod Status
Alerts
```

---

# Lab Steps

# Step 1 - Create Grafana Namespace

Create:

```bash
touch yaml/lab-38/grafana-namespace.yaml
```

Add:

```yaml
apiVersion: v1

kind: Namespace


metadata:

  name: monitoring
```

---

Apply:

```bash
kubectl apply -f yaml/lab-38/grafana-namespace.yaml
```

---

# Step 2 - Create Grafana Deployment

Create:

```bash
touch yaml/lab-38/grafana-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: grafana

  namespace: monitoring


spec:

  replicas: 1


  selector:

    matchLabels:

      app: grafana


  template:

    metadata:

      labels:

        app: grafana


    spec:

      containers:

      - name: grafana

        image: grafana/grafana

        ports:

        - containerPort: 3000
```

---

Apply:

```bash
kubectl apply -f yaml/lab-38/grafana-deployment.yaml
```

---

# Step 3 - Create Grafana Service

Create:

```bash
touch yaml/lab-38/grafana-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: grafana

  namespace: monitoring


spec:

  type: NodePort


  selector:

    app: grafana


  ports:

  - port: 3000

    targetPort: 3000
```

---

Apply:

```bash
kubectl apply -f yaml/lab-38/grafana-service.yaml
```

---

# Step 4 - Verify Grafana Pod

Run:

```bash
kubectl get pods -n monitoring
```

Expected:

```
grafana-xxxxx    Running
```

---

# Step 5 - Access Grafana

Get URL:

```bash
minikube service grafana -n monitoring --url
```

Open the URL.

Grafana login page appears.

---

# Step 6 - Login Grafana

Default credentials:

Username:

```
admin
```

Password:

```
admin
```

Grafana will ask to change password.

---

# Step 7 - Add Prometheus Data Source

Inside Grafana:

Go to:

```
Connections
```

Then:

```
Data Sources
```

Select:

```
Prometheus
```

URL:

```
http://prometheus:9090
```

Save and Test.

Expected:

```
Data source is working
```

---

# Step 8 - Create Dashboard

Go to:

```
Dashboards

Create Dashboard

Add Visualization
```

Select Prometheus.

Query example:

```
up
```

Save dashboard.

---

# Useful Prometheus Queries

## Check Targets

```
up
```

---

## CPU Usage

```
process_cpu_seconds_total
```

---

## Memory Usage

```
process_resident_memory_bytes
```

---

# Kubernetes Dashboard Examples

Grafana can show:

## Cluster Overview

- Nodes
- CPU
- Memory

## Pod Monitoring

- Pod status
- Restart count
- Resource usage

## Application Monitoring

- Request count
- Response time

---

# Debug Commands

Check Grafana:

```bash
kubectl get pods -n monitoring
```

---

Check Logs:

```bash
kubectl logs POD_NAME -n monitoring
```

---

Check Service:

```bash
kubectl get svc -n monitoring
```

---

Describe Pod:

```bash
kubectl describe pod POD_NAME -n monitoring
```

---

# Verification

Run:

```bash
kubectl get pods -n monitoring
```

Expected:

```
prometheus     Running
grafana        Running
```

Open Grafana:

```
Dashboard loads successfully
```

---

# Troubleshooting Checklist

## Grafana Pod Not Running

Check:

```bash
kubectl describe pod POD_NAME -n monitoring
```

---

## Cannot Connect Prometheus

Verify:

```bash
kubectl get svc -n monitoring
```

Check URL:

```
http://prometheus:9090
```

---

## Dashboard Empty

Check:

```
Prometheus datasource
```

Verify Prometheus has metrics.

---

# Interview Questions

## 1. What is Grafana?

Grafana is a visualization platform used to create dashboards from metrics.

---

## 2. Difference between Prometheus and Grafana?

Prometheus:

Collects and stores metrics.

Grafana:

Displays metrics.

---

## 3. How does Grafana get Kubernetes metrics?

Using Prometheus as a data source.

---

## 4. Default Grafana port?

```
3000
```

---

## 5. Why use dashboards?

To quickly understand system health.

---

# Commands Learned

```bash
kubectl get pods -n monitoring

kubectl logs

kubectl describe pod

kubectl get svc

minikube service
```

---

# Cleanup

Delete Grafana:

```bash
kubectl delete deployment grafana -n monitoring
```

Delete Service:

```bash
kubectl delete svc grafana -n monitoring
```

---

# Navigation

Previous:

[Lab 37 - Prometheus Monitoring](lab-37-prometheus.md)

Next:

[Lab 39 - Three Tier Application](lab-39-three-tier-app.md)

---

# Lab Completed ✅

You learned:

- Grafana
- Prometheus integration
- Kubernetes dashboards
- Monitoring visualization