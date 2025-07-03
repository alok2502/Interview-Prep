### ❓ Question: Explain the architecture of a Kubernetes cluster

### ✅ Answer:

The architecture of a Kubernetes cluster can be divided into two main parts:

### 🔹 Control Plane (Master Node)

This is responsible for the overall management of the cluster. Its main components are:

1. **API Server** – This is the **entry point** for all the Kubernetes REST commands. Every request (like `kubectl apply`) first hits the API server.
2. **Scheduler** – When a new pod needs to be deployed, the API server passes the request to the scheduler, which decides the **best node** to place the pod on.
3. **etcd** – Acts as the **key-value store** (like a database) for all cluster data, including configuration, state, and metadata.
4. **Controller Manager** – It manages all the controllers in Kubernetes. For example, the **ReplicaSet controller** ensures the desired number of pod replicas are running at any time.

### 🔹 Data Plane (Worker Node)

This is where actual application workloads (containers) run. Key components include:

1. **kubelet** – An agent that runs on each worker node. It ensures that containers described in PodSpecs are running and healthy.
2. **Container Runtime** – The software (like containerd, Docker) responsible for **running the actual containers**. The kubelet interacts with this runtime to start/stop containers.
3. **kube-proxy** – Handles **networking** in the cluster. It maintains network rules on nodes, allowing communication between pods across nodes.

---

### 🔁 Overall Flow of Pod Deployment:

1. A user runs a `kubectl apply -f pod.yaml` command.
2. The request goes to the **API Server**.
3. The **Scheduler** selects the most appropriate worker node.
4. The **API Server** then instructs the **kubelet** on that node to run the pod.
5. The **kubelet**, in turn, uses the **Container Runtime** to start the container.
6. **kube-proxy** ensures that networking is in place for communication.
7. All this state information is stored in **etcd**.
8. The **Controller Manager** monitors the system and ensures the desired state (e.g., number of replicas) is maintained.

---

💬 This architectural understanding shows the **clear separation of responsibilities** between the control plane and the data plane and how Kubernetes ensures high availability and self-healing.
