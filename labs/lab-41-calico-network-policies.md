# Phase 1 – Create Kubernetes Cluster Without CNI

### Step 1.1 – Check Minikube

```bash
minikube version
docker --version
kubectl version --client
minikube profile list
```

### Step 1.2 – Start Minikube

Create a cluster with three nodes and no CNI:

```bash
minikube start --profile soft --nodes 3 --driver=docker --cni=false --network-plugin=cni --extra-config=kubeadm.pod-network-cidr=192.168.0.0/16 --subnet=172.16.0.0/24
```

### Step 1.3 – Check current context

```bash
kubectl config current-context
```

Expected:

```text
soft
```

### Step 1.4 – Check nodes

```bash
kubectl get nodes
```

Without CNI, nodes may show NotReady.

# Phase 2 – Install Calico CNI

We will use Calico v3.33.0 with the Tigera Operator.

### Step 2.1 – Check Kubernetes version

```bash
kubectl version
```

For Kubernetes 1.36 or later, run:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/v3_projectcalico_org.yaml
```

For Kubernetes 1.34 or 1.35, use:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/v3_projectcalico_org-v1beta1.yaml
```

Choose only the manifest matching your Kubernetes version.

### Step 2.2 – Install Tigera Operator

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/tigera-operator.yaml
```

### Step 2.3 – Install Calico

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.33.0/manifests/custom-resources.yaml
```

The default pod CIDR in this manifest is 192.168.0.0/16, matching our cluster configuration.

### Step 2.4 – Verify Calico

```bash
kubectl get pods -n tigera-operator
```

```bash
kubectl get pods -n calico-system
```

```bash
kubectl get tigerastatus
```

### Step 2.5 – Verify nodes

```bash
kubectl get nodes -o wide
```

Expected: all three nodes eventually become Ready.

# Phase 3 – Deploy Three NGINX Pods

We will create three independent NGINX pods in the default namespace.

### Step 3.1 – Create pod.yaml

Go inside the pods folder:

```bash
cd pods
```

Create `pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    app: nginx
    role: pod1
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      ports:
        - containerPort: 80

---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    app: nginx
    role: pod2
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      ports:
        - containerPort: 80

---
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  labels:
    app: nginx
    role: pod3
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      ports:
        - containerPort: 80
```

### Step 3.2 – Apply YAML file

```bash
kubectl apply -f pod.yaml
```

### Step 3.3 – Verify pods

```bash
kubectl get pods -o wide
```

Expected:

```text
NAME    READY   STATUS
pod1    1/1     Running
pod2    1/1     Running
pod3    1/1     Running
```

Check labels:

```bash
kubectl get pods --show-labels
```

### Step 3.4 – Understand our pod labels

| Pod | Label |
|---|---|
| pod1 | role=pod1 |
| pod2 | role=pod2 |
| pod3 | role=pod3 |

We will use these labels inside our Network Policies.

# Phase 4 – Baseline Connectivity Test

### Objective

Before applying Network Policies, verify that all three pods can communicate with each other.

At this stage, no Network Policies exist, so pod traffic is allowed by default.

### Step 4.1 – Get Pod IP addresses

```bash
kubectl get pods -o wide
```

Example:

```text
NAME    IP
pod1    10.244.0.5
pod2    10.244.0.6
pod3    10.244.0.7
```

Your IP addresses may be different.

### Step 4.2 – Test Pod 2 to Pod 1

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://10.244.0.5
```

Replace the IP with your actual Pod 1 IP.

Expected: NGINX welcome page HTML.

### Step 4.3 – Test Pod 1 to Pod 2

```bash
kubectl exec pod1 -- wget -T 3 -qO- http://10.244.0.6
```

### Step 4.4 – Test Pod 1 to Pod 3

```bash
kubectl exec pod1 -- wget -T 3 -qO- http://10.244.0.7
```

### Step 4.5 – Test Pod 2 to Pod 3

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://10.244.0.7
```

Expected: All tests succeed.

**Why are we using wget instead of curl?**

The `nginx:alpine` image includes BusyBox wget, so we can test connectivity without installing additional tools.

We are using Pod IPs instead of DNS names because DNS will be blocked in our later default-deny egress phase.

# Phase 5 – Block All Pod Communication

### Objective

Now we will block all incoming and outgoing traffic for all pods.

We will create a default-deny Network Policy.

### Step 5.1 – Create 01-default-deny-all.yaml

Go to the project root:

```bash
cd ..
cd policies
```

Create `01-default-deny-all.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny-all

spec:
  podSelector: {}

  policyTypes:
    - Ingress
    - Egress
```

### Step 5.2 – Understand the YAML

| Field | Explanation |
|---|---|
| kind: NetworkPolicy | Creates a network policy |
| podSelector: {} | Selects all pods |
| policyTypes: Ingress | Blocks incoming traffic unless allowed |
| policyTypes: Egress | Blocks outgoing traffic unless allowed |

**Important:** An empty `podSelector: {}` selects every pod in the policy's namespace.

### Step 5.3 – Apply the policy

```bash
kubectl apply -f 01-default-deny-all.yaml
```

### Step 5.4 – Verify

```bash
kubectl get networkpolicy
```

### Step 5.5 – Test Pod 2 to Pod 1

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://10.244.0.5
```

Expected: Connection blocked or timed out.

Test the other directions too.

| Source | Destination | Expected |
|---|---|---|
| Pod 1 | Pod 2 | Blocked |
| Pod 1 | Pod 3 | Blocked |
| Pod 2 | Pod 1 | Blocked |
| Pod 2 | Pod 3 | Blocked |
| Pod 3 | Pod 1 | Blocked |
| Pod 3 | Pod 2 | Blocked |

**What happened?**

We blocked both Ingress and Egress traffic for all three pods.

# Phase 6 – Allow Only Pod 1 to Communicate with Pod 2

### Objective

Now we will allow only this connection:

```text
Pod 1  -------->  Pod 2
 ALLOWED
```

All other connections must remain blocked.

### Step 6.1 – Create 02-allow-pod1-to-pod2.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: allow-pod1-egress-to-pod2

spec:
  podSelector:
    matchLabels:
      role: pod1

  policyTypes:
    - Egress

  egress:
    - to:
        - podSelector:
            matchLabels:
              role: pod2
      ports:
        - protocol: TCP
          port: 80

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: allow-pod1-ingress-to-pod2

spec:
  podSelector:
    matchLabels:
      role: pod2

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: pod1
      ports:
        - protocol: TCP
          port: 80
```

### Step 6.2 – Understand the policy

The first policy:

- Selects Pod 1.
- Allows outgoing traffic.
- Allows Pod 1 to connect to Pod 2.
- Only allows TCP port 80.

The second policy:

- Selects Pod 2.
- Allows incoming traffic.
- Only accepts traffic from Pod 1.
- Only allows TCP port 80.

We need both policies because the source must be allowed to send and the destination must be allowed to receive.

### Step 6.3 – Apply the policy

```bash
kubectl apply -f 02-allow-pod1-to-pod2.yaml
```

### Step 6.4 – Test allowed connection

```bash
kubectl exec pod1 -- wget -T 3 -qO- http://10.244.0.6
```

Expected: Success. NGINX welcome page.

### Step 6.5 – Test blocked connections

Pod 2 to Pod 1:

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://10.244.0.5
```

Pod 1 to Pod 3:

```bash
kubectl exec pod1 -- wget -T 3 -qO- http://10.244.0.7
```

Pod 2 to Pod 3:

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://10.244.0.7
```

Expected: All three connections fail.

# Phase 7 – Create a New Network Policy

### Objective

Now we want to establish this communication:

```text
Pod 1  -------->  Pod 2  -------->  Pod 3
 ALLOWED           ALLOWED
```

But we do not want:

```text
Pod 1  -------->  Pod 3     BLOCKED

Pod 2  -------->  Pod 1     BLOCKED
```

### Step 7.1 – Important: Remove the previous allow policy

Network Policies are additive. The previous allow policy remains active unless deleted.

Keep `default-deny-all`, but remove the Phase 6 allow rules:

```bash
kubectl delete -f 02-allow-pod1-to-pod2.yaml
```

Now all three pods are blocked again.

### Step 7.2 – Create 03-allow-pod1-pod2-pod3.yaml

This file contains four policy objects.

```yaml
# Allow Pod 1 to send traffic to Pod 2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: pod1-egress-to-pod2
spec:
  podSelector:
    matchLabels:
      role: pod1
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: pod2
      ports:
        - protocol: TCP
          port: 80

---
# Allow Pod 2 to receive traffic from Pod 1
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: pod2-ingress-from-pod1
spec:
  podSelector:
    matchLabels:
      role: pod2
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: pod1
      ports:
        - protocol: TCP
          port: 80

---
# Allow Pod 2 to send traffic to Pod 3
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: pod2-egress-to-pod3
spec:
  podSelector:
    matchLabels:
      role: pod2
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: pod3
      ports:
        - protocol: TCP
          port: 80

---
# Allow Pod 3 to receive traffic from Pod 2
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: pod3-ingress-from-pod2
spec:
  podSelector:
    matchLabels:
      role: pod3
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: pod2
      ports:
        - protocol: TCP
          port: 80
```

### Step 7.3 – Apply the policy

```bash
kubectl apply -f 03-allow-pod1-pod2-pod3.yaml
```

### Step 7.4 – Verify policies

```bash
kubectl get networkpolicy
```

# Phase 8 – Final Connectivity Testing

Get the latest Pod IP addresses:

```bash
kubectl get pods -o wide
```

Use your actual Pod IPs in the following tests.

### Test 1 – Pod 1 to Pod 2

```bash
kubectl exec pod1 -- wget -T 3 -qO- http://POD2_IP
```

Expected: SUCCESS.

### Test 2 – Pod 2 to Pod 3

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://POD3_IP
```

Expected: SUCCESS.

### Test 3 – Pod 1 to Pod 3

```bash
kubectl exec pod1 -- wget -T 3 -qO- http://POD3_IP
```

Expected: BLOCKED.

### Test 4 – Pod 2 to Pod 1

```bash
kubectl exec pod2 -- wget -T 3 -qO- http://POD1_IP
```

Expected: BLOCKED.

### Final test result

| Source | Destination | Expected |
|---|---|---|
| Pod 1 | Pod 2 | SUCCESS |
| Pod 2 | Pod 3 | SUCCESS |
| Pod 1 | Pod 3 | BLOCKED |
| Pod 2 | Pod 1 | BLOCKED |
| Pod 3 | Pod 1 | BLOCKED |
| Pod 3 | Pod 2 | BLOCKED |

**Lab success condition:** Only Pod 1 → Pod 2 and Pod 2 → Pod 3 are allowed.

# Phase 9 – Cleanup

Delete the Network Policies:

```bash
kubectl delete networkpolicy --all
```

Delete the three pods:

```bash
kubectl delete -f pods/pod.yaml
```

Delete the Minikube cluster:

```bash
minikube delete -p soft
```

# Interview Questions

**1. What is CNI?**

CNI provides network connectivity between Kubernetes pods.

**2. What is Calico?**

Calico is a CNI plugin that provides pod networking and network security policies.

**3. What is a Network Policy?**

A Network Policy controls incoming and outgoing traffic between pods.

**4. What is Ingress?**

Ingress is incoming traffic to a pod.

**5. What is Egress?**

Egress is outgoing traffic from a pod.

**6. What does podSelector: {} mean?**

It selects all pods in the namespace.

**7. What is default deny?**

It blocks traffic unless explicitly allowed by another applicable policy.

**8. Can we allow communication between two specific pods?**

Yes. We use podSelector with matching labels.

**9. Do we need both Ingress and Egress rules for Pod 1 to communicate with Pod 2?**

When both pods are isolated in both directions, yes. We must allow Pod 1's Egress and Pod 2's Ingress.

**10. Are Network Policies applied in order?**

No. Their allow rules are additive.

---

## Important references

- Kubernetes Network Policies: https://kubernetes.io/docs/concepts/services-networking/network-policies/
- Calico Minikube installation: https://docs.tigera.io/calico/latest/getting-started/kubernetes/minikube
- Calico Network Policy tutorial: https://docs.tigera.io/calico/latest/network-policy/get-started/kubernetes-policy/kubernetes-network-policy