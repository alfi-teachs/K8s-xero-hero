# LAB 29 - Kubernetes Job

## Objective

In this lab, you will learn:

- What is a Kubernetes Job
- Difference between Pod and Job
- Create a Job
- Monitor Job execution
- Understand completions and retries
- Run batch workloads

---

# What is a Kubernetes Job?

A Kubernetes Job creates Pods and ensures that a task completes successfully.

Unlike Deployments:

Deployment:

```
Application runs continuously
```

Example:

```
Web server
Database
API
```

Job:

```
Task runs and finishes
```

Example:

```
Database backup

Report generation

Data processing
```

---

# Real World Example

Imagine an e-commerce company.

Every night:

```
12:00 AM

Backup database
```

The task:

```
Start

Run backup

Finish

Exit
```

This is a Job.

---

# Job Flow

```
User

 |

Job Object

 |

Creates Pod

 |

Container Runs

 |

Task Completes

 |

Pod Completed

```

---

# Job vs Deployment

| Job | Deployment |
|---|---|
| Runs once | Runs continuously |
| Stops after completion | Keeps Pods running |
| Batch processing | Applications |
| Backup jobs | Web applications |

---

# Job Components

## completions

Number of successful executions required.

Example:

```yaml
completions: 3
```

Means:

```
Run task 3 times successfully
```

---

## parallelism

Number of Pods running at the same time.

Example:

```yaml
parallelism: 2
```

Means:

```
Run 2 Pods together
```

---

## backoffLimit

Number of retries if Job fails.

Example:

```yaml
backoffLimit: 3
```

Means:

```
Retry 3 times
```

---

# Lab Steps

# Step 1 - Create Simple Job

Create:

```bash
touch yaml/lab-29/simple-job.yaml
```

Add:

```yaml
apiVersion: batch/v1

kind: Job


metadata:

  name: hello-job


spec:

  template:

    spec:

      containers:

      - name: hello

        image: busybox

        command:

        - sh

        - -c

        - echo "Hello Kubernetes Job"


      restartPolicy: Never
```

---

# Step 2 - Create Job

Run:

```bash
kubectl apply -f yaml/lab-29/simple-job.yaml
```

---

# Step 3 - Check Job

Run:

```bash
kubectl get jobs
```

Expected:

```
NAME

hello-job

COMPLETIONS

1/1
```

---

# Step 4 - Check Pod

Run:

```bash
kubectl get pods
```

Expected:

```
hello-job-xxxxx

Completed
```

---

# Step 5 - View Job Logs

Find Pod:

```bash
kubectl get pods
```

Get logs:

```bash
kubectl logs POD_NAME
```

Output:

```
Hello Kubernetes Job
```

---

# Step 6 - Create Job With Multiple Completions

Create:

```bash
touch yaml/lab-29/multi-job.yaml
```

Add:

```yaml
apiVersion: batch/v1

kind: Job


metadata:

  name: multi-job


spec:

  completions: 3

  parallelism: 2


  template:

    spec:

      containers:

      - name: worker

        image: busybox

        command:

        - sh

        - -c

        - echo "Processing task"


      restartPolicy: Never
```

---

Apply:

```bash
kubectl apply -f yaml/lab-29/multi-job.yaml
```

---

# Step 7 - Watch Job Execution

Run:

```bash
kubectl get jobs -w
```

Expected:

```
NAME

multi-job

COMPLETIONS

3/3
```

---

# Step 8 - Check Completed Pods

Run:

```bash
kubectl get pods
```

You will see:

```
Completed
Completed
Completed
```

---

# Step 9 - Describe Job

Run:

```bash
kubectl describe job hello-job
```

Check:

```
Events
```

---

# Step 10 - Delete Job

Run:

```bash
kubectl delete job hello-job
```

---

# Verification

Check Jobs:

```bash
kubectl get jobs
```

Check Pods:

```bash
kubectl get pods
```

Check logs:

```bash
kubectl logs POD_NAME
```

---

# Troubleshooting

## Job Failed

Check:

```bash
kubectl describe job JOB_NAME
```

---

## Pod Failed

Check:

```bash
kubectl describe pod POD_NAME
```

---

## View Container Error

Run:

```bash
kubectl logs POD_NAME
```

---

# Interview Questions

## 1. What is Kubernetes Job?

A Job creates Pods and ensures a task completes successfully.

---

## 2. Difference between Job and Deployment?

Job:

Runs tasks and stops.

Deployment:

Runs applications continuously.

---

## 3. What is completions?

Number of successful Pod executions required.

---

## 4. What is parallelism?

Number of Pods allowed to run simultaneously.

---

## 5. What is backoffLimit?

Maximum number of retries after failure.

---

# Commands Learned

```bash
kubectl get jobs

kubectl describe job

kubectl logs

kubectl get pods

kubectl delete job
```

---

# Cleanup

Delete Jobs:

```bash
kubectl delete job hello-job
```

```bash
kubectl delete job multi-job
```

---

# Navigation

Previous:

[Lab 28 - ClusterRole and ClusterRoleBinding](lab-28-clusterrole.md)

Next:

[Lab 30 - Kubernetes CronJob](lab-30-cronjob.md)

---

# Lab Completed ✅

You learned:

- Kubernetes Jobs
- Batch processing
- completions
- parallel execution
- Job troubleshooting