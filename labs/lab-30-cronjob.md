# LAB 30 - Kubernetes CronJob

## Objective

In this lab, you will learn:

- What is Kubernetes CronJob
- Difference between Job and CronJob
- Understand Cron schedules
- Create scheduled tasks
- Monitor CronJob executions
- Run automatic batch workloads

---

# What is Kubernetes CronJob?

A CronJob creates Jobs automatically based on a schedule.

It is similar to Linux cron.

Example:

Linux:

```
Run backup every night at 12 AM
```

Kubernetes:

```
CronJob
   |
   |
Creates Job
   |
   |
Creates Pod
   |
   |
Task completes
```

---

# Real World Examples

CronJobs are used for:

## Database Backup

Every night:

```
12:00 AM

Backup database
```

---

## Report Generation

Every morning:

```
8:00 AM

Generate sales report
```

---

## Cleanup Tasks

Every week:

```
Remove old files
```

---

# Job vs CronJob

| Job | CronJob |
|---|---|
| Runs once | Runs on schedule |
| Manual execution | Automatic execution |
| One-time task | Repeated task |
| Backup now | Backup every day |

---

# Cron Expression

CronJob uses five fields:

```
Minute Hour Day Month Weekday
```

Example:

```
0 0 * * *
```

Means:

```
Every day at midnight
```

---

# Cron Examples

Every minute:

```
* * * * *
```

Every hour:

```
0 * * * *
```

Every day at 12 AM:

```
0 0 * * *
```

Every Sunday:

```
0 0 * * 0
```

---

# CronJob Flow

```
CronJob

   |

Schedule Time Reached

   |

Creates Job

   |

Creates Pod

   |

Task Runs

   |

Pod Completed

```

---

# Lab Steps

# Step 1 - Create CronJob YAML

Create:

```bash
touch yaml/lab-30/backup-cronjob.yaml
```

Add:

```yaml
apiVersion: batch/v1

kind: CronJob


metadata:

  name: backup-cronjob


spec:

  schedule: "*/1 * * * *"


  jobTemplate:

    spec:

      template:

        spec:

          containers:

          - name: backup

            image: busybox

            command:

            - sh

            - -c

            - echo "Database backup completed"


          restartPolicy: Never
```

---

# Step 2 - Create CronJob

Run:

```bash
kubectl apply -f yaml/lab-30/backup-cronjob.yaml
```

Expected:

```
cronjob.batch/backup-cronjob created
```

---

# Step 3 - Check CronJob

Run:

```bash
kubectl get cronjobs
```

Output:

```
NAME

backup-cronjob
```

---

# Step 4 - Wait For Job Creation

Because schedule is every minute:

Run:

```bash
kubectl get jobs -w
```

You will see:

```
backup-cronjob-xxxxx
```

Press:

```
CTRL + C
```

---

# Step 5 - Check Pods

Run:

```bash
kubectl get pods
```

Example:

```
backup-cronjob-xxxxx

Completed
```

---

# Step 6 - Check Logs

Get Pod name:

```bash
kubectl get pods
```

View logs:

```bash
kubectl logs POD_NAME
```

Output:

```
Database backup completed
```

---

# Step 7 - Describe CronJob

Run:

```bash
kubectl describe cronjob backup-cronjob
```

Check:

```
Schedule

Last Schedule Time

Events
```

---

# Step 8 - Create Manual Job From CronJob

Run:

```bash
kubectl create job manual-backup --from=cronjob/backup-cronjob
```

Check:

```bash
kubectl get jobs
```

---

# Step 9 - Delete CronJob

Run:

```bash
kubectl delete cronjob backup-cronjob
```

---

# Verification

Check CronJobs:

```bash
kubectl get cronjobs
```

Check Jobs:

```bash
kubectl get jobs
```

Check Pods:

```bash
kubectl get pods
```

---

# Troubleshooting

## CronJob Not Creating Jobs

Check:

```bash
kubectl describe cronjob backup-cronjob
```

Look at:

```
Events
```

---

## Job Failed

Check:

```bash
kubectl describe job JOB_NAME
```

View logs:

```bash
kubectl logs POD_NAME
```

---

# Interview Questions

## 1. What is CronJob in Kubernetes?

CronJob runs Jobs automatically according to a schedule.

---

## 2. Difference between Job and CronJob?

Job:

Runs once.

CronJob:

Creates Jobs repeatedly based on schedule.

---

## 3. Which format does CronJob use?

Cron expression.

Example:

```
0 0 * * *
```

---

## 4. What creates Pods in CronJob?

Flow:

```
CronJob
 |
Job
 |
Pod
```

---

## 5. Where are CronJobs used?

Examples:

- Backups
- Reports
- Cleanup tasks
- Scheduled batch processing

---

# Commands Learned

```bash
kubectl get cronjobs

kubectl describe cronjob

kubectl get jobs

kubectl create job --from=cronjob

kubectl logs
```

---

# Cleanup

Delete CronJob:

```bash
kubectl delete cronjob backup-cronjob
```

Delete Jobs:

```bash
kubectl delete jobs --all
```

---

# Navigation

Previous:

[Lab 29 - Kubernetes Job](lab-29-job.md)

Next:

[Lab 31 - CrashLoopBackOff Troubleshooting](lab-31-crashloopbackoff.md)

---

# Lab Completed ✅

You learned:

- Kubernetes CronJob
- Cron schedules
- Automated tasks
- Scheduled batch processing