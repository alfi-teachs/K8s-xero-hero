# LAB 37 - Prometheus Monitoring

## Objective

In this lab, you will learn:

- What monitoring means in Kubernetes
- What Prometheus is
- Prometheus architecture
- How Prometheus collects metrics
- Deploy Prometheus in Kubernetes
- Query Kubernetes metrics

---

# What is Monitoring?

Monitoring means continuously checking the health and performance of applications and infrastructure.

Example metrics:

CPU usage

```
70%
```

Memory usage

```
500MB / 1GB
```

Number of Pods

```
5 Running Pods
```

---

# Why Kubernetes Monitoring?

In production environments we need to know:

- Is application running?
- Is CPU increasing?
- Is memory full?
- Are Pods failing?
- Is traffic increasing?

---

# What is Prometheus?

Prometheus is an open-source monitoring system.

It collects:

- CPU metrics
- Memory metrics
- Network metrics
- Application metrics
- Kubernetes object metrics

---

# Prometheus Architecture

```
                Kubernetes Cluster

                       |

                 Prometheus Server

                       |

              Metrics Collection

                       |

        +--------------+--------------+

        |              |              |

     Nodes          Pods          Services

```

---

# Prometheus Components

## 1. Prometheus Server

Main component.

Responsibilities:

- Collect metrics
- Store metrics
- Query metrics

---

## 2. Exporters

Exporters expose metrics.

Examples:

Node Exporter:

```
Node CPU
Node Memory
```

kube-state-metrics:

```
Pod status
Deployment status
Service information
```

---

## 3. Alertmanager

Sends alerts.

Example:

```
CPU > 90%

Send Email Alert
```

---

# How Prometheus Works

Flow:

```
Application

     |

Metrics Endpoint

     |

Prometheus Scrapes Metrics

     |

Stores Time Series Data

     |

User Queries Data

```

---

# Kubernetes Monitoring Stack

Common stack:

```
Prometheus

+

Grafana

+

Alertmanager

+

Exporters
```

---

# Lab Steps

# Step 1 - Create Monitoring Namespace

Create:

```bash
touch yaml/lab-37/namespace.yaml
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
kubectl apply -f yaml/lab-37/namespace.yaml
```

Check:

```bash
kubectl get namespaces
```

---

# Step 2 - Create Prometheus ConfigMap

Create:

```bash
touch yaml/lab-37/prometheus-config.yaml
```

Add:

```yaml
apiVersion: v1

kind: ConfigMap


metadata:

  name: prometheus-config

  namespace: monitoring


data:

  prometheus.yml: |

    global:

      scrape_interval: 15s


    scrape_configs:

    - job_name: kubernetes-pods

      static_configs:

      - targets:

        - localhost:9090
```

---

Apply:

```bash
kubectl apply -f yaml/lab-37/prometheus-config.yaml
```

---

# Step 3 - Create Prometheus Deployment

Create:

```bash
touch yaml/lab-37/prometheus-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1

kind: Deployment


metadata:

  name: prometheus

  namespace: monitoring


spec:

  replicas: 1


  selector:

    matchLabels:

      app: prometheus


  template:

    metadata:

      labels:

        app: prometheus


    spec:

      containers:

      - name: prometheus

        image: prom/prometheus


        ports:

        - containerPort: 9090


        volumeMounts:

        - name: config

          mountPath: /etc/prometheus


      volumes:

      - name: config

        configMap:

          name: prometheus-config
```

---

Apply:

```bash
kubectl apply -f yaml/lab-37/prometheus-deployment.yaml
```

---

# Step 4 - Create Prometheus Service

Create:

```bash
touch yaml/lab-37/prometheus-service.yaml
```

Add:

```yaml
apiVersion: v1

kind: Service


metadata:

  name: prometheus

  namespace: monitoring


spec:

  type: NodePort


  selector:

    app: prometheus


  ports:

  - port: 9090

    targetPort: 9090
```

---

Apply:

```bash
kubectl apply -f yaml/lab-37/prometheus-service.yaml
```

---

# Step 5 - Verify Prometheus

Check Pods:

```bash
kubectl get pods -n monitoring
```

Expected:

```
prometheus-xxxxx   Running
```

---

Check Service:

```bash
kubectl get svc -n monitoring
```

---

# Step 6 - Access Prometheus

Get URL:

```bash
minikube service prometheus -n monitoring --url
```

Open URL in browser.

You should see:

```
Prometheus Web UI
```

---

# Step 7 - Query Metrics

In Prometheus search:

```
up
```

Result:

```
1
```

Means Prometheus is running.

---

# Important Prometheus Queries

## Check Running Targets

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

# Debug Commands

Check Namespace:

```bash
kubectl get ns
```

Check Pods:

```bash
kubectl get pods -n monitoring
```

Check Logs:

```bash
kubectl logs POD_NAME -n monitoring
```

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

Prometheus should show:

```
Running
```

Access:

```
Prometheus Web UI
```

Query:

```
up
```

---

# Troubleshooting Checklist

If Prometheus is not working:

## 1. Check Pod

```bash
kubectl get pods -n monitoring
```

---

## 2. Check Logs

```bash
kubectl logs POD_NAME -n monitoring
```

---

## 3. Check ConfigMap

```bash
kubectl describe configmap prometheus-config -n monitoring
```

---

## 4. Check Service

```bash
kubectl get svc -n monitoring
```

---

# Interview Questions

## 1. What is Prometheus?

Prometheus is an open-source monitoring and alerting system.

---

## 2. How does Prometheus collect metrics?

Using a pull model called scraping.

---

## 3. What is an exporter?

A component that exposes metrics for Prometheus.

---

## 4. What is Alertmanager?

A component that sends notifications when alerts occur.

---

## 5. Difference between Prometheus and Grafana?

Prometheus:

Collects and stores metrics.

Grafana:

Visualizes metrics using dashboards.

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

Delete monitoring resources:

```bash
kubectl delete namespace monitoring
```

---

# Navigation

Previous:

[Lab 36 - Network Troubleshooting](lab-36-network-troubleshooting.md)

Next:

[Lab 38 - Grafana Monitoring](lab-38-grafana.md)

---

# Lab Completed ✅

You learned:

- Kubernetes monitoring
- Prometheus architecture
- Metrics collection
- Prometheus deployment