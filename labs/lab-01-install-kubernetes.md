# LAB 01 - Install Kubernetes

## Objective

In this lab, you will learn:

- What Kubernetes is
- Kubernetes architecture
- Difference between Control Plane and Worker Node
- Install kubectl
- Install Minikube
- Start a Kubernetes cluster
- Verify Kubernetes cluster health
- Use basic kubectl commands

## What You Will Learn
By the end of this lab, you will understand:

- What Kubernetes does
- Kubernetes Cluster
- Control Plane components
- Worker Node components
- How kubectl communicates with Kubernetes
- How to create and verify a local Kubernetes cluster using Minikube
---
# What is Kubernetes?

Kubernetes is an open-source container orchestration platform.

It helps us:

- Deploy applications
- Manage containers
- Restart failed containers automatically
- Scale applications
- Manage networking
- Manage storage

Example:

Imagine you have an online shopping website.

Your application has:

- Frontend container
- Backend container
- Database container

If one container crashes, Kubernetes automatically creates a new one.
## Real World Example

Imagine you have an online shopping website.

Your application contains:

- Frontend container
- Backend container
- Database container

If a container crashes, Kubernetes automatically detects the failure and creates a new container.

Kubernetes acts as a manager that maintains the desired state of your application.
---

# Kubernetes Architecture

             Kubernetes Cluster

                   |
                   |

          +----------------+
          |  Control Plane |
          +----------------+

                   |
                   |

          +----------------+
          |  Worker Nodes  |
          +----------------+
---

# Control Plane (Master Node)

The Control Plane manages the Kubernetes cluster.

Main components:

## API Server

The API Server is the communication point between users and Kubernetes.

Example:

When we run:

```bash
kubectl get pods
```
kubectl sends the request to the API Server.

Example:

```bash
User
 |
 |
kubectl
 |
 |
API Server
 |
 |
Kubernetes Cluster
```
### Scheduler

Scheduler decides:

"On which worker node should this Pod run?"

Example:
```bash
New Pod
   |
   |
Scheduler
   |
   |
Worker Node 1
```
The Scheduler checks available resources and selects the best worker node.

### Controller Manager

Controller Manager watches the desired state.

Example:

You create:

replicas: 3

If one Pod fails:

Desired Pods = 3

Running Pods = 2

Controller creates a new Pod.

### etcd

etcd stores Kubernetes cluster information.

It stores:

Nodes
Pods
Services
Configurations
# Worker Node

Worker Nodes run the actual applications.

Each Worker Node contains:

### Kubelet

Kubelet communicates with the Control Plane.

Responsibilities:

Receive instructions from API Server
Create containers
Monitor container health
### Container Runtime

Container Runtime runs containers.

Examples:

Docker
containerd
### kube-proxy

kube-proxy manages network communication.

It allows:

Pod-to-Pod communication
Service-to-Pod communication

## Prerequisites

Install:
```bash
Docker Desktop
kubectl
Minikube
Git
```

## Lab Steps
### Step 1 - Install kubectl

Verify kubectl installation:
```bash
kubectl version --client
```
Expected:

Client Version: v1.xx.x
### Step 2 - Install Minikube

Verify Minikube:
```bash
minikube version
```
Expected:

minikube version: v1.xx.x
### Step 3 - Start Kubernetes Cluster

Start Minikube:
```bash
minikube start
```
Expected:

Done! kubectl is now configured to use "minikube"
### Step 4 - Check Minikube Status

Command:
```bash
minikube status
```
Expected:

host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
### Step 5 - Verify Kubernetes Cluster

Check cluster information:
```bash
kubectl cluster-info
```
Example:

Kubernetes control plane is running
### Step 6 - Check Nodes

Command:
```bash
kubectl get nodes
```
Expected:

NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane   5m

Explanation:

Ready
 |
 Kubernetes node is healthy
### Step 7 - Check Kubernetes Namespaces

Command:
```bash
kubectl get namespaces
```
Expected:

default
kube-system
kube-public
kube-node-lease
### Step 8 - Check Running System Pods

Command:
```bash
kubectl get pods -n kube-system
```
You will see Kubernetes internal components.

Example:

coredns
kube-apiserver
kube-controller-manager
kube-scheduler
### Step 9 - Basic kubectl Commands
Check Nodes
```bash
kubectl get nodes
```
Get Pods
```bash
kubectl get pods
```
Get All Resources
```bash
kubectl get all
```
## Verification

View Cluster Information
```bash
kubectl cluster-info
```
Get Kubernetes Version
```bash
kubectl version
```
## Troubleshooting
### Problem 1: Minikube is not starting

Check status:
```bash
minikube status
```
Restart cluster:
```bash
minikube stop
```
```bash
minikube start
```
### Problem 2: kubectl Cannot Connect

Check current context:
```bash
kubectl config current-context
```
Expected:

minikube

Change context:
```bash
kubectl config use-context minikube
```
## Interview Questions

1. What is Kubernetes?

Kubernetes is a container orchestration platform used to deploy and manage containers.

2. What is a Kubernetes Cluster?

A cluster is a group of machines running Kubernetes.

3. Difference between Control Plane and Worker Node?

Control Plane manages the cluster.

Worker Nodes run applications.

4. What does kubelet do?

Kubelet ensures containers are running on worker nodes.

## Commands Learned
Troubleshooting
Minikube is not starting

Check:
```bash
minikube status
```
Restart:
```bash
minikube stop
minikube start
```
kubectl cannot connect

Check context:
```bash
kubectl config current-context
```
Expected:

minikube

Change context:
```bash
kubectl config use-context minikube
```
### Lab Verification

Run:
```bash
minikube status
kubectl get nodes
kubectl get pods -n kube-system
```
All components should be running.

## Cleanup
Cleanup

Stop Minikube:
```bash
minikube stop
```
Delete cluster:
```bash
minikube delete
```
## Navigation
Navigation

Previous:

None (First Lab)

Next:
labs/lab-02-first-pod.md
