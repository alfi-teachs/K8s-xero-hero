# LAB 01 - Install Kubernetes

## Objective

## What You Will Learn
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

---

# Kubernetes Architecture

A Kubernetes cluster has two main parts:
Kubernetes Cluster

    |
    |

+-------------------+
| Control Plane |
+-------------------+

    |
    |

+-------------------+
| Worker Nodes |
+-------------------+


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
kubectl talks to the API Server.
Scheduler

Scheduler decides:

"On which worker node should this Pod run?"

Example:

New Pod
   |
   |
Scheduler
   |
   |
Worker Node 1
Controller Manager

Controller Manager watches the desired state.

Example:

You create:

replicas: 3

If one Pod fails:

Desired Pods = 3

Running Pods = 2

Controller creates a new Pod.

etcd

etcd stores Kubernetes cluster information.

It stores:

Nodes
Pods
Services
Configurations
Worker Node

Worker nodes run the actual applications.

Components:

Kubelet

Kubelet communicates with the Control Plane.

It ensures containers are running.

Container Runtime

Runs containers.

Examples:

Docker
containerd
kube-proxy

Handles network communication between Pods and Services.
## Prerequisites
Prerequisites

Install:

Docker Desktop
kubectl
Minikube
Git
## Theory

## Architecture

## Lab Steps
Step 1 - Install kubectl

Verify kubectl installation:

kubectl version --client

Expected:

Client Version: v1.xx.x
Step 2 - Install Minikube

Verify Minikube:

minikube version

Expected:

minikube version: v1.xx.x
Step 3 - Start Kubernetes Cluster

Start Minikube:

minikube start

Expected:

Done! kubectl is now configured to use "minikube"
Step 4 - Check Minikube Status

Command:

minikube status

Expected:

host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
Step 5 - Verify Kubernetes Cluster

Check cluster information:

kubectl cluster-info

Example:

Kubernetes control plane is running
Step 6 - Check Nodes

Command:

kubectl get nodes

Expected:

NAME       STATUS   ROLES           AGE
minikube   Ready    control-plane   5m

Explanation:

Ready
 |
 Kubernetes node is healthy
Step 7 - Check Kubernetes Namespaces

Command:

kubectl get namespaces

Expected:

default
kube-system
kube-public
kube-node-lease
Step 8 - Check Running System Pods

Command:

kubectl get pods -n kube-system

You will see Kubernetes internal components.

Example:

coredns
kube-apiserver
kube-controller-manager
kube-scheduler
Step 9 - Basic kubectl Commands
Check Nodes
kubectl get nodes
Get Pods
kubectl get pods
Get All Resources
kubectl get all
View Cluster Information
kubectl cluster-info
Get Kubernetes Version
kubectl version

## Verification

## Troubleshooting

## Interview Questions
nterview Questions
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

minikube status

Restart:

minikube stop
minikube start
kubectl cannot connect

Check context:

kubectl config current-context

Expected:

minikube

Change context:

kubectl config use-context minikube
Lab Verification

Run:

minikube status
kubectl get nodes
kubectl get pods -n kube-system

All components should be running.

## Cleanup
Cleanup

Stop Minikube:

minikube stop

Delete cluster:

minikube delete
## Navigation
