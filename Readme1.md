# Kubernetes One Shot
---

### **K8s History and Why Should We Learn K8s**

#### **History:**
- Kubernetes was developed by Google in 2014 as an open-source project, based on their internal system called **Borg**.
- It became the de-facto standard for container orchestration, hosted under the **Cloud Native Computing Foundation (CNCF)**.

#### **Why Learn K8s?**
- Automates the deployment, scaling, and management of containerized applications.
- **Key Benefits**:
  - **High Availability:** Handles failover, self-healing.
  - **Scalability:** Dynamically adjusts application resources.
  - **Portability:** Runs anywhere (on-premises or cloud).

---

### **Monolithic vs Microservices**

- **Monolithic Architecture**: All functionalities are bundled into a single unit.
  - **Example**: A single web server handling authentication, database interactions, and UI rendering.
- **Microservices Architecture**: Breaks the application into smaller services, each responsible for a specific functionality.
  - **Example**: Independent services for login, orders, and notifications.

**Comparison Table**:
| Feature           | Monolithic          | Microservices       |
|--------------------|---------------------|---------------------|
| Scalability        | Limited             | Highly Scalable     |
| Development Speed  | Slower              | Faster              |
| Fault Isolation    | Difficult           | Easier              |

---

### **K8s Architecture**
Kubernetes uses a **master-worker architecture**:
1. **Control Plane (Master Node)**:
   - **API Server**: Handles REST requests.
   - **Controller Manager**: Ensures the desired state.
   - **Scheduler**: Schedules pods on nodes.
   - **etcd**: Stores the cluster state.

2. **Worker Nodes**:
   - **Kubelet**: Manages pods.
   - **Kube-proxy**: Manages networking.
   - **Container Runtime**: Docker or containerd.

---

### **Kubectl**
**Kubectl** is the CLI for Kubernetes.

**Basic Syntax**:
```bash
kubectl <command> <resource> <name> [flags]
```

**Example**:
```bash
kubectl get pods            # List all pods
kubectl describe pod mypod  # Detailed pod info
kubectl delete pod mypod    # Delete a pod
```

---

### **Create K8s Architecture by Own**
To build your own architecture:
1. Install **Docker**.
2. Set up a control plane using tools like **Kubeadm**, **Minikube**, or **KIND**.
3. Deploy nodes to form a cluster.

---

### **What is KIND Cluster & Installation**
- **KIND (Kubernetes IN Docker)** is a tool to create K8s clusters using Docker containers.
- It’s lightweight and useful for local development/testing.

#### **Installation**:
```bash
# Install KIND
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

---

### **KIND Setup on Local**
Create a configuration file:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
```

**Create the cluster**:
```bash
kind create cluster --config kind-config.yaml
```

---

### **What is Minikube Cluster, Installation, Cluster Creation**
- **Minikube** runs a single-node Kubernetes cluster locally.

#### **Installation**:
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

#### **Cluster Creation**:
```bash
minikube start
kubectl get nodes
```

---

### **Kubeadm, Setup & Installation, Cluster Creation**
**Kubeadm** initializes a K8s cluster by installing all necessary components.

#### **Installation**:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

#### **Cluster Initialization**:
```bash
sudo kubeadm init
```

---

### **Namespaces**
Namespaces isolate resources within a cluster.

**Example**:
```bash
kubectl create namespace dev
kubectl get namespaces
```
**Example**:
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: nginx
```
---

### **Pods**
A pod is the smallest deployable unit in Kubernetes.

**Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

**Deploy**:
```bash
kubectl apply -f pod.yaml
```

---

### **Deployments**
Deployments manage the desired state of pods.

**Example**:
```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      name: nginx-dep-pod
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Deploy**:
```bash
kubectl apply -f deployment.yaml
```

---

### **ReplicaSet vs StatefulSet vs Deployment**

| Feature        | ReplicaSet             | StatefulSet          | Deployment         |
|----------------|------------------------|----------------------|--------------------|
| Use Case       | Maintain Pod count     | Stateful apps        | Updates & Scaling  |
| Pod Naming     | Random                 | Sequential           | Random             |
| Scaling        | Easy                   | Complex              | Easy               |

---

### **Labels and Selectors**
Labels identify resources; selectors group them.

**Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: nginx
  labels:
    env: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

**Selector**:
```bash
kubectl get pods --selector env=dev
```

---

### **Rolling Update in Deployment**
Ensures no downtime during updates.

**Command**:
```bash
kubectl set image deployment/nginx-deploymen -n nginx nginx=nginx:latest
kubectl set image deployment/nginx-deployment -n nginx nginx=nginx:1.27.3
```

---

### **ReplicaSets**
Maintains the desired number of replicas.

**Example**:
```yaml
kind: ReplicaSet
apiVersion: apps/v1
metadata:
  name: nginx-replicaset
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      name: nginx-rep-pod
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

### **DaemonSets**
Runs one pod per node.

**Example**:
```yaml
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: nginx-daemonsets
  namespace: nginx
spec:
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      name: nginx-dmn-pod
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

### **Jobs & CronJobs**
Jobs: Executes tasks to completion.
CronJobs: Scheduled jobs.

**Example (CronJob)**:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "0 12 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: myjob
            image: busybox
            args:
            - /bin/sh
            - -c
            - echo "Hello Kubernetes!"
          restartPolicy: OnFailure
```

**Deploy**:
```bash
kubectl apply -f cronjob.yaml
```

---

This covers the topics in a detailed yet approachable manner, with examples and syntax! Let me know if you need further clarification or more topics expanded.
