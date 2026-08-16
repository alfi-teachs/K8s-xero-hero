# Kubernetes Level 1 — Quick Q&A

## 1. What is a Pod?
A Pod is the smallest deployable unit in Kubernetes and runs one or more containers.

---

## 2. What is a Deployment?
Deployment manages Pods through ReplicaSets and handles:
- Scaling
- Rolling updates
- Rollbacks

Flow:

Deployment → ReplicaSet → Pods

---

## 3. What does a ReplicaSet do?
Maintains the desired number of Pods.

If 3 replicas are required and 1 Pod dies:

ReplicaSet → creates a new Pod.

---

## 4. What is a Service?
Provides a stable way to access Pods.

Main types:
- ClusterIP → internal
- NodePort → external through node port
- LoadBalancer → external cloud access

---

## 5. How does a Service find Pods?
Using labels and selectors.

Pod:

app: frontend

Service:

selector:
  app: frontend

They must match.

---

## 6. What happens if Service selector doesn't match Pod labels?
Service gets no matching endpoints.

Check:

kubectl get endpoints <service-name>

---

## 7. Does NodePort traffic have to reach a Pod on the same node?
No.

Example:

User → node2:30080 → Service → Pod on node1

---

## 8. What is Service port vs targetPort?
port = Service port

targetPort = Application/container port

Example:

port: 80
targetPort: 8080

---

## 9. What is a Namespace?
Logical separation of Kubernetes resources.

Example:

development
production
testing

---

## 10. What is ConfigMap?
Stores non-sensitive configuration.

Example:
- PORT
- HOST
- ENV

---

## 11. What is a Secret?
Stores sensitive data such as:
- Passwords
- Tokens
- API keys

Secret data is commonly Base64 encoded.

IMPORTANT:
Base64 ≠ encryption.

---

## 12. What is Readiness Probe?
Checks:

"Is the Pod ready to receive traffic?"

If readiness fails:

Pod stays running but Service stops sending traffic to it.

---

## 13. What is Liveness Probe?
Checks:

"Is the application still alive?"

If liveness repeatedly fails:

Container is restarted.

---

## 14. What is Startup Probe?
Gives a slow-starting application time to start.

Startup succeeds → Liveness + Readiness take over.

---

## 15. Running vs Ready?
Running does NOT always mean Ready.

Example:

READY: 0/1
STATUS: Running

Container is running but not ready for traffic.

---

## 16. What is a CPU request?
The amount of CPU Kubernetes considers when scheduling a Pod.

500m = 0.5 CPU

---

## 17. What is a CPU limit?
Maximum CPU the container is allowed to use.

1 CPU = 1000m

---

## 18. What is HPA?
HPA = Horizontal Pod Autoscaler.

It increases/decreases the number of Pods based on metrics.

Example:

2 Pods → 3 → 4 → 5

HPA does NOT increase CPU inside an existing Pod.

---

## 19. What does Metrics Server do?
Provides CPU and memory usage metrics.

Used by:

kubectl top pods
kubectl top nodes
HPA

---

## 20. What is a Taint?
Taint is applied to a NODE to keep Pods away.

Example:

dedicated=db:NoSchedule

---

## 21. What is a Toleration?
Toleration is applied to a POD and allows it to run on a matching tainted node.

IMPORTANT:

Toleration allows.
Toleration does NOT force.

---

## 22. What are the three taint effects?

NoSchedule
→ Don't schedule new non-tolerating Pods.

PreferNoSchedule
→ Try to avoid the node.

NoExecute
→ Don't schedule new Pods AND evict existing non-tolerating Pods.

---

## 23. What is nodeSelector?
Tells a Pod to run only on nodes with a specific label.

Node:

disk=ssd

Pod:

nodeSelector:
  disk: ssd

---

## 24. What is Node Affinity?
Provides more flexible node selection.

required = MUST match

preferred = TRY to match

Remember:

required = MUST
preferred = TRY

---

## 25. What is Pod Affinity?
Places a Pod near other matching Pods.

"I want to be near these Pods."

---

## 26. What is Pod Anti-Affinity?
Keeps Pods away from other matching Pods.

Useful for high availability.

"I don't want my replicas on the same node."

---

## 27. What is a Rolling Update?
Gradually replaces old Pods with new Pods.

Example:

nginx:1.25 → nginx:1.26

Deployment uses:
Old ReplicaSet + New ReplicaSet

Eventually:

Old ReplicaSet → 0 Pods
New ReplicaSet → desired replicas

---

## 28. How do you check rollout status?

kubectl rollout status deployment/nginx

---

## 29. How do you see rollout history?

kubectl rollout history deployment/nginx

---

## 30. How do you rollback?

Previous version:

kubectl rollout undo deployment/nginx

Specific revision:

kubectl rollout undo deployment/nginx --to-revision=2

---

## 31. How do you change a Deployment image?

kubectl set image deployment/web nginx=nginx:1.26

---

# IMPORTANT MEMORY RULES

Deployment → ReplicaSet → Pods

Taint → Node
Toleration → Pod

Label → identifies object
Selector → finds matching objects

Readiness → Traffic
Liveness → Restart
Startup → Slow startup

Request → Scheduling
Limit → Maximum resource

HPA → Number of Pods

required → MUST
preferred → TRY

Pod Affinity → Near
Pod Anti-Affinity → Away

NodePort → Entry through node
Service → Finds Pods using selector

Base64 → Encoding, NOT encryption

Running ≠ Ready