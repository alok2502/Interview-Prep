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

---

### ❓ Question: How do various components of Kubernetes interact with each other?

### ✅ Answer:

When we run a `kubectl` command to deploy a resource (e.g., a Deployment), here's how the interaction between Kubernetes components happens:

1. The **API Server** (on the control plane) receives the request from `kubectl`. It is the central communication hub for all components.
2. The API Server then passes the scheduling task to the **Scheduler**, which decides the most appropriate node (e.g., worker-node3) for deploying the pods.
3. Once a node is selected, the API Server contacts the **kubelet** on that node. The kubelet is an agent that runs on every node and ensures that the containers described in the manifest are running properly.
4. The **kubelet** interacts with the **Container Runtime** (like Docker or containerd) to pull the container image and start the container.
5. Meanwhile, **kube-proxy** ensures that network rules are applied and the pod can communicate with other pods or services as defined.
6. All state and metadata related to this deployment is stored in **etcd**, the key-value store of the cluster.
7. Finally, the **Controller Manager** continuously watches the cluster state and ensures that controllers (like ReplicaSet) are fulfilling their responsibilities — such as maintaining the desired number of pod replicas.

💡 This interaction ensures that Kubernetes is able to deploy, scale, heal, and maintain applications in a reliable and automated way.

---

### ❓ Question: What is the use of services in Kubernetes?

### ✅ Answer:

In Kubernetes, a **Service** is an abstraction that enables stable networking and communication between pods and external users. It mainly provides three core functionalities:

1. **Service Discovery** – Pods are ephemeral and can get new IPs if restarted. Services solve this by using **labels and selectors** to dynamically match and route traffic to the correct pods, regardless of their IP changes.

2. **Load Balancing** – Services distribute incoming network traffic **evenly across all healthy pod replicas**, usually using a **round-robin algorithm**.

3. **Exposure** – Services allow us to expose applications either **within the cluster** or **to the outside world**, depending on the type of service (ClusterIP, NodePort, LoadBalancer, etc).

💡 Without services, communication between pods or exposing applications to users would be unstable and error-prone.

---

### ❓ Question: Why is hardcoding Pod IP communication a bad practice in Kubernetes?

### ✅ Answer:

Hardcoding Pod IPs for communication in Kubernetes is a bad practice because **Pods are ephemeral in nature**, meaning they can be recreated at any time due to crashes, rescheduling, or updates — and when that happens, they usually get **a new IP address**.

For example, suppose you have two pods:

* **Pod A** runs the frontend
* **Pod B** runs the backend

If Pod A has Pod B’s IP address hardcoded for backend communication, and Pod B restarts, Pod A will continue trying to connect to the **old IP**, which no longer exists. This leads to broken connectivity.

💡 To solve this, Kubernetes uses **Services**, which act as a stable communication layer. A Service uses **selectors and labels** to dynamically route traffic to the current set of matching pods, regardless of their underlying IPs. This ensures reliable and maintainable inter-pod communication.

---

### ❓ Question: What are the different types of Services in Kubernetes?

### ✅ Answer:

Kubernetes supports different types of Services to meet various networking needs. The main types are:

1. **ClusterIP (default)** –

   * Exposes the service **internally within the cluster**.
   * It’s not accessible from outside the cluster.
   * Commonly used for internal service-to-service communication.

2. **NodePort** –

   * Exposes the service **on a static port on each node’s IP address**.
   * Allows **external access** by hitting `NodeIP:NodePort`.
   * Mostly used for simple setups or local testing.

3. **LoadBalancer** –

   * Works with **cloud providers** that support Load Balancers.
   * Automatically provisions an **external load balancer** and assigns a **public IP**.
   * Requires **Cloud Controller Manager (CCM)** support.

4. **Headless Service (ClusterIP: None)** –

   * Skips the default ClusterIP assignment.
   * Used when you want direct access to individual pod IPs — often used with **StatefulSets** or **custom DNS-based service discovery**.

5. **ExternalName** –

   * Maps a service name to an **external DNS name**.
   * No selector or endpoints are involved — acts like a DNS alias.

💡 Most interviews focus on the first three, but mentioning the last two shows deeper awareness. If you're not confident in their usage, just mention them briefly and say you’ve read about them.

---

### ❓ Question: What are labels and selectors in Kubernetes?

### ✅ Answer:

In Kubernetes:

* **Labels** are **key-value pairs** attached to resources like Pods, Nodes, and Services. They are used to **organize, group, and filter** Kubernetes objects. For example:

  ```yaml
  labels:
    app: frontend
    env: production
  ```

* **Selectors** are used by Kubernetes components like **Services, ReplicaSets, and Deployments** to **identify and interact with specific Pods** based on their labels.

There are two types of selectors:

1. **Equality-based** – e.g., `app = frontend`
2. **Set-based** – e.g., `env in (production, staging)`

💡 This mechanism decouples the configuration and allows dynamic targeting of resources — crucial for scalability and flexibility.

---

### ❓ Question: What would you recommend — NodePort or LoadBalancer — and why?

### ✅ Answer:

Both **NodePort** and **LoadBalancer** are types of Kubernetes Services used to expose applications, but they serve different use cases:

* **NodePort** exposes the application on a static port on each node’s IP address. It is typically used when you want to access the application **within a private network or VPC**. Anyone with access to the node IP and NodePort can reach the service.

* **LoadBalancer** is used when you need to expose your application to the **public internet**. It provisions a **public IP** and routes external traffic to the underlying nodes/services. This type of service is only available in **cloud platforms that support external load balancers**, such as AWS, GCP, or Azure. It requires a **Cloud Controller Manager (CCM)** integration.

#### ✅ Recommendation:

* Use **NodePort** for internal or restricted access within the VPC or on-prem environments.
* Use **LoadBalancer** when the application must be publicly accessible and you are operating in a supported cloud environment.

💡 Note: LoadBalancers are more user-friendly but also incur **additional cost** and are not available in bare-metal clusters without extra setup.

### ❓ Question: How are Kubernetes Services related to kube-proxy?

### ✅ Answer:

Kubernetes Services play a key role in service discovery. Let’s say we have two pods — Pod A (frontend) and Pod B (backend). If Pod A tries to reach Pod B using its IP address, it may fail because pods are **ephemeral** — they can be recreated with different IPs at any time. This makes direct communication using pod IPs unreliable.

To solve this, Kubernetes provides **Services**, which act as stable virtual IPs and rely on **label selectors** to target matching pods. This abstraction ensures that even if pods go down and come back with new IPs, the Service still routes traffic correctly.

Now, here’s where **kube-proxy** comes in:
When a Service is created, Kubernetes also creates an **Endpoints** object that stores the IPs of the pods matched by the Service. Kube-proxy watches for changes in Services and Endpoints and configures the underlying **iptables** or **IPVS** rules on each node. These rules ensure that any request to the Service IP is forwarded to one of the healthy pod IPs behind it.

💡 In short, kube-proxy is responsible for implementing the Service networking by setting up rules to load-balance traffic to the appropriate backend pods.

---

### ❓ Question: What are the disadvantages of using a Service of type LoadBalancer in Kubernetes?

### ✅ Answer:

The main disadvantage of using a **LoadBalancer** Service type in Kubernetes is the **cost and scalability** overhead in cloud environments.

When you create a LoadBalancer service on platforms like **Amazon EKS**, **Google GKE**, or **Azure AKS**, it provisions a **dedicated external load balancer** (e.g., an **AWS ALB or ELB**) and assigns it a **public IP address**.

However, this comes with some limitations:

1. **One Load Balancer per Service** – Every time you create a new LoadBalancer-type service, the cloud provider creates a **new external load balancer**. These are **not shared or reused** between services. If you expose five services, five separate load balancers will be created, which significantly increases infrastructure **costs**.

2. **Dependency on Cloud Provider Integration** – LoadBalancer services require the **Cloud Controller Manager (CCM)** and proper cloud provider support. They are not available in **bare-metal clusters** unless extra setup is done (e.g., with MetalLB).

3. **Limited Customization** – You have limited control over the configuration of the cloud provider’s load balancer through native Kubernetes YAML alone.

💡 **Better Alternative:**
To reduce cost and manage multiple services with a **single public IP**, it's common to use an **Ingress controller** (like NGINX or AWS ALB Ingress), which can route traffic to multiple services using **path-based** or **host-based** routing.

### ❓ Question  
How would you restrict access to a database so that only one specific pod can access it within the same namespace?

---

### ✅ Answer  
To restrict access to a database so that **only one pod** (e.g., a specific application or microservice) can access it within the same namespace, we use a **NetworkPolicy**. Here's how:

---

#### 🔹 Step-by-Step Approach

1. **Label the database pod** – Add a unique label to identify the database pod:
   ```yaml
   labels:
     app: db
   ```

2. **Label the allowed application pod** – Label the application/microservice pod that should be allowed to access the database:
   ```yaml
   labels:
     app: myapp
   ```

3. **Create a NetworkPolicy** – Define a policy with `Ingress` type that only allows traffic from pods with a specific label (`app: myapp`) to the database pod.

   Example:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: allow-myapp-to-db
   spec:
     podSelector:
       matchLabels:
         app: db  # target the database pod
     policyTypes:
       - Ingress
     ingress:
       - from:
           - podSelector:
               matchLabels:
                 app: myapp  # only allow this pod
   ```

---

💡 This ensures that no other pod in the namespace (or cluster) can talk to the database except the one explicitly allowed. You can even narrow it further by restricting ports or namespaces if needed.

### ❓ Question: What deployment strategy do you follow in your organization?

### ✅ Answer:

In our organization, we follow the **Canary deployment model** for releasing new application versions.

Instead of updating all pods at once, we begin by routing a small percentage of traffic (e.g., 50%) to the new version. If it performs well without errors, we gradually increase the traffic share until it reaches 100%. This approach allows us to minimize risk and catch issues early before they impact all users.

As a DevOps engineer, I typically follow these steps:

1. **Create a new version of the application** with updated pods.
2. **Set up a new Service and Ingress** specifically for the new version.
3. In the **Ingress annotations**, I define routing rules — for example, start with 50% of traffic going to the new service and 50% to the old one.
4. Monitor the new version's performance and gradually update the routing to move 100% of the traffic to the new service.
5. Once verified, we decommission the old version's resources.

💡 This strategy gives us fine-grained control over rollout and rollback, which is especially valuable in production environments where stability is critical.

### ❓ Question: What rollback strategy do you follow in your organization?

### ✅ Answer:

Since we use the **Canary deployment model**, the risk of a full-scale failure is already minimized by initially routing only a portion of traffic to the new version. However, if a bug still slips through and we need to roll back, we follow a **GitOps-based rollback strategy**.

Here’s how we handle it:

1. All our Kubernetes manifests and Helm charts are version-controlled in a **Git repository**.
2. We use **ArgoCD** with **auto-sync enabled** to continuously deploy changes from Git to our Kubernetes clusters.
3. If a rollback is needed, we simply **change the Helm chart version** in the Git repository to point to the **previous stable version**.
4. ArgoCD detects this change and **automatically reverts the deployment** in the cluster — rolling back the pods to the previous image version.

💡 Because we follow GitOps, the rollback is declarative, trackable via Git history, and doesn't require any manual kubectl commands — making it safe and auditable.

### ❓ Question: How do you design a deployment strategy that avoids the need for rollbacks?

### ✅ Answer:

In our organization, we aim to **avoid full rollbacks altogether** by using a combination of **Canary deployments** and **GitOps principles**.

#### 🔹 Canary Deployment to Minimize Impact:

Instead of deploying the new version to 100% of users, we first release it to a **small percentage** (e.g., 10–50%). This helps us identify bugs or performance issues early without affecting all users. If everything looks good, we gradually increase traffic to the new version.

#### 🔹 GitOps for Safe and Auditable Control:

Even in rare cases where rollback is necessary, we follow a **GitOps-based strategy**:

- All Kubernetes manifests and Helm charts are stored in Git.
- We use **ArgoCD** with **auto-sync** enabled.
- If something goes wrong, we simply **update the Helm chart version** in the Git repo to point to the previous stable release.
- ArgoCD picks up the change and automatically rolls the cluster back to the earlier version.

💡 This approach allows us to **detect issues early**, limit blast radius, and avoid rollbacks in most cases — while still keeping rollback easy, fast, and auditable when needed.

### ❓ Question: What deployment strategies have you used in the past?

### ✅ Answer:

In my current organization, I primarily work with the **Canary deployment model**. In this approach, we gradually shift traffic to the new version of the application instead of routing all users at once. This helps us catch potential bugs or issues early, without impacting all users.

### 🔹 How we use Canary:
- Deploy the new version and route only a small percentage (e.g., 10–50%) of traffic to it.
- Monitor metrics and logs.
- If everything looks good, gradually shift traffic to 100%.
- If any issues are found, we use a GitOps-based rollback using ArgoCD and Helm.

I'm also familiar with the **Blue-Green deployment strategy**, although I haven’t used it extensively in production. In Blue-Green:
- We maintain two environments: the current stable one (**Blue**) and the new version (**Green**).
- Once the new version is tested and verified, all traffic is switched to Green.
- If any issues occur, traffic is switched back to the Blue environment (the previous stable version).

💡 Both strategies aim to reduce downtime and risk during deployment. I prefer Canary for its gradual rollout and fine-grained traffic control.

### ❓ Question  
What is the role of CoreDNS in Kubernetes?

---

### ✅ Answer  
**CoreDNS** is the default **DNS server** used in Kubernetes for **service discovery**. It translates **DNS names into IP addresses**, allowing pods to communicate with each other using **domain names** rather than hardcoded IPs.

---

### 🔹 Example Use Case  
Let’s say Pod A wants to talk to a backend service called `payment`. In the pod’s configuration, you might see:

```
payment.default.svc.cluster.local
```

This is a **fully qualified domain name (FQDN)** for the `payment` service in the `default` namespace. CoreDNS is responsible for **resolving** this domain name to the actual **ClusterIP** of the service, for example: `20.10.10.3`.

---

### 🔹 Why it's Important  
- **Pods are ephemeral**, and their IPs can change. Services expose a stable DNS name.  
- CoreDNS ensures pods can always reach services reliably, regardless of underlying IP changes.  
- It watches the Kubernetes API to stay updated with new Services and Endpoints.

---

💡 Without CoreDNS, inter-pod and pod-to-service communication using names like `my-service.default.svc.cluster.local` wouldn’t work.

### ❓ Question  
A DevOps engineer has tainted a node with `NoSchedule`. Can we still deploy an application on that node?

---

### ✅ Answer  
Yes, we **can still deploy** an application on a tainted node, but only if the pod has a **matching toleration**.

When a node is tainted using a command like:

```bash
kubectl taint nodes node1 key=app:NoSchedule
```

…it means that no pods will be scheduled on that node unless they explicitly tolerate the taint.

To allow a pod to run on this node, we add a toleration in the pod spec:

```yaml
tolerations:
  - key: "app"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

---

### 🔹 Summary  
- **Taint** = repel all pods that don’t tolerate it.  
- **Toleration** = opt-in to allow scheduling on a tainted node.

---

💡 This is commonly used to dedicate certain nodes to specific workloads, like system-level pods or critical services.

### ❓ Question  
A pod is stuck in CrashLoopBackOff. What steps would you take to troubleshoot it?

---

### ✅ Answer  
**CrashLoopBackOff** is a state in Kubernetes where a pod starts, crashes, and keeps restarting in a loop. To troubleshoot it, I follow these steps:

---

### 🔹 1. Check Pod Logs  
Use the `kubectl logs` command to view logs from the crashing container:

```bash
kubectl logs <pod-name> --previous
```

The `--previous` flag helps when the current container has already restarted.

Look for:
- Application errors  
- Missing environment variables  
- Unhandled exceptions  

---

### 🔹 2. Describe the Pod  
If logs don’t help, run:

```bash
kubectl describe pod <pod-name>
```

This command shows:
- Events (e.g., failed image pulls, missing secrets, etc.)  
- Liveness/readiness probe failures  
- Node-level issues or taints  

---

💡 These steps usually pinpoint the root cause—once identified, you can fix the pod spec, image config, or readiness probe as needed.

### ❓ Question: What is the difference between an Ingress and a Service of type LoadBalancer in Kubernetes?

### ✅ Answer:

Both **Ingress** and **LoadBalancer-type Services** are used to expose applications to the **public internet**, but they differ in architecture, cost, and flexibility.

### 🔹 Service of type LoadBalancer:

- Directly provisions a **cloud provider’s external load balancer** (like AWS ELB/ALB) via the **Cloud Controller Manager**.
- Each LoadBalancer service creates a **dedicated load balancer** for that one service.
- This can become **expensive**, especially if multiple services need public access.
- Simpler to set up but **less flexible** for routing.

### 🔹 Ingress:

- Requires an **Ingress Controller** (like NGINX, Traefik, or AWS ALB Ingress).
- You only need **one load balancer** (behind the Ingress controller) to **expose multiple services**.
- Ingress allows **path-based** and **host-based routing** (e.g., `/api` goes to one service, `/app` to another).
- **More cost-effective** and customizable than using multiple LoadBalancer services.

### 🔁 Summary:

| Feature              | LoadBalancer Service           | Ingress                           |
|----------------------|-------------------------------|------------------------------------|
| Public Access        | Yes                           | Yes                                |
| Cost                 | High (one LB per service)     | Low (one LB can serve many routes) |
| Routing Flexibility  | Basic                         | Path/Host-based                    |
| Cloud Dependency     | Yes (Cloud Controller needed) | Yes (but more portable)            |

💡 Use **Ingress** when managing multiple services behind a single endpoint, and use **LoadBalancer** when simplicity or dedicated access is required.

### ❓ Question  
Your application works with ClusterIP but fails with Ingress. How would you troubleshoot this issue?

---

### ✅ Answer  
If the application is accessible via a **ClusterIP Service** but fails when accessed through an **Ingress**, the issue is likely in the Ingress configuration or controller setup. Here’s how I’d troubleshoot it:

---

### 🔹 1. Verify Service and Port Mapping  
- Check that the **Ingress resource correctly references the Service**.  
- Make sure the **backend Service name and port** are correctly defined in the Ingress rules.  
- Confirm that the Service itself is reachable and functioning as expected.

---

### 🔹 2. Confirm Ingress Controller Is Deployed  
- Many people forget to deploy the **Ingress Controller** (like NGINX, Traefik, or AWS ALB Ingress).  
- The Ingress resource alone does nothing unless a controller is actively watching and configuring the necessary routes or load balancers.

You can check with:
```bash
kubectl get pods -n ingress-nginx
```

Or for AWS:
```bash
kubectl get pods -n kube-system
```

---

### 🔹 3. Inspect Ingress Controller Logs  
If the controller is running, the next step is to check its logs for routing errors, TLS issues, or misconfigured rules.

```bash
kubectl logs <ingress-controller-pod-name> -n <namespace>
```

---

### 🔹 4. Validate DNS and Annotations  
- Ensure DNS is pointing to the ingress controller’s external IP.  
- Check that all required Ingress annotations are correctly set — especially for specific controllers (e.g., ALB, GKE).

---

💡 Ingress debugging often comes down to missing controllers, misconfigured paths, or incorrect service links. Start from the service layer and work upward toward the controller and external access.

### ❓ Question: Why do I need to set up an Ingress Controller if I’ve already created an Ingress resource?

### ✅ Answer:

Creating an **Ingress resource** in Kubernetes defines the desired rules for routing traffic (like path-based or host-based routing), but **by itself**, it doesn’t actually handle the traffic.

Just like a **Deployment** relies on the **ReplicaSet controller** to create pods, an **Ingress resource needs an Ingress Controller** to:

1. **Watch for Ingress resources** in the cluster.
2. **Provision or configure the actual Load Balancer** (in cloud setups) or update reverse proxy rules (in NGINX, Traefik, etc).
3. **Route incoming HTTP/HTTPS traffic** based on Ingress rules to the correct Services/pods.

### 🔹 Without an Ingress Controller:

- The Ingress resource just sits there — no traffic will be routed.
- No LoadBalancer or routes are created automatically.

### 🔹 With an Ingress Controller:

- It watches the API for new Ingress rules.
- Configures routes and load balancing based on those rules.
- Ensures HTTP traffic from the outside world is properly forwarded to backend services.

💡 Think of the Ingress resource as the *blueprint* and the Ingress Controller as the *builder* that makes it work.

### ❓ Question  
Your Deployment has 3 replicas, but only one pod is running. What could be the reason?

---

### ✅ Answer  
If only 1 out of 3 replicas is running, it usually indicates a **scheduling issue** or a **resource constraint**. Here's how I’d troubleshoot it:

---

### 🔹 1. Check Deployment Configuration  
Use:
```bash
kubectl describe deployment <deployment-name>
```
- Look at the **Events** section for errors related to pod creation or scheduling.  
- See if the Deployment is reporting something like `1/3 replicas available`.

---

### 🔹 2. Investigate Resource Requests  
- If your Deployment has CPU or memory requests, Kubernetes can only schedule pods on nodes that meet those resource requirements.  
- If nodes don’t have enough resources, the scheduler won’t place the pods.

---

### 🔹 3. Check Node Conditions and Taints  
Run:
```bash
kubectl get nodes
kubectl describe nodes
```
- Verify if some nodes are tainted and whether your pods have the necessary tolerations.  
- Also check if any nodes are in a `NotReady` or `SchedulingDisabled` state.

---

### 🔹 4. Not Enough Nodes Available  
If resources are low or taints block scheduling, Kubernetes may not have enough capacity to run all replicas. In that case:
- Scale the cluster by adding more nodes.  
- Or optimize resource requests/limits in your Deployment.

---

💡 The key is that the Deployment controller is trying, but the Kube Scheduler can't place the pods due to constraints.

### ❓ Question  
Your pod mounts a ConfigMap, but changes to the ConfigMap are not getting reflected. Why?

---

### ✅ Answer  
There are **two ways** to inject ConfigMap data into a pod:  
1. As **environment variables**  
2. As **mounted volumes**

---

### 🔹 If Using Environment Variables  
- When a ConfigMap is used as **environment variables**, the values are **loaded only once when the pod starts**.  
- **Changes to the ConfigMap won’t reflect** in the pod until it is **manually restarted**.

```yaml
env:
  - name: CONFIG_MODE
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: mode
```

---

### 🔹 If Using Volume Mounts  
When the ConfigMap is mounted as a volume, Kubernetes keeps the mounted file in sync with the ConfigMap contents.  
Changes to the ConfigMap will be automatically reflected inside the pod—usually within a few seconds.

```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
volumes:
  - name: config-volume
    configMap:
      name: my-config
```

---

### 💡 Summary  

| Method           | Auto-Reflect Changes? | Requires Pod Restart? |
|------------------|-----------------------|------------------------|
| Env Variables    | ❌ No                 | ✅ Yes                 |
| Volume Mount     | ✅ Yes                | ❌ No                 |

---

So, if you're seeing no updates, check how the ConfigMap is being consumed in the pod.

### ❓ Question: What is Node Affinity in Kubernetes?

### ✅ Answer:

**Node Affinity** is a feature in Kubernetes used to **influence pod scheduling** based on **node labels**. It allows us to control which nodes a pod is eligible to run on — similar to `nodeSelector`, but **more powerful and flexible**.

---

### 🔹 Why It's Used:

- To ensure certain pods run only on specific types of nodes.
- For example, pods that require GPUs should be scheduled only on GPU-enabled nodes.

---

### 🔹 Types of Node Affinity:

1. **RequiredDuringSchedulingIgnoredDuringExecution** (Hard affinity):
   - Pod **must** be scheduled on a node matching the rule.
   - If no matching node is found, the pod won’t be scheduled.

2. **PreferredDuringSchedulingIgnoredDuringExecution** (Soft affinity):
   - Pod **prefers** to be scheduled on matching nodes, but if not possible, it can go elsewhere.

---

### 🔹 Example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

This tells Kubernetes to only place the pod on nodes labeled `disktype=ssd`.

💡 `nodeSelector` is like basic filtering. Node Affinity gives you expression-based matching, which is more flexible and production-ready.

---

### ❓ Question: What is the difference between Node Affinity and Node Label Selector in Kubernetes?

### ✅ Answer:

Both **Node Affinity** and **Node Label Selector** are used to control which nodes a pod can be scheduled on, but Node Affinity is a more advanced and flexible version of node label selectors.

---

### 🔹 Node Label Selector (`nodeSelector`):

- Basic way to schedule pods onto nodes based on exact match of labels.
- Only supports equality-based rules (e.g., `disktype=ssd`).

**Example:**

```yaml
nodeSelector:
  disktype: ssd
```

It’s a hard requirement — the pod will only run on matching nodes.

---

### 🔹 Node Affinity:

- Introduced as a more expressive alternative to `nodeSelector`.
- Supports:
  - **Hard Affinity** (`requiredDuringSchedulingIgnoredDuringExecution`) – must run on matching node.
  - **Soft Affinity** (`preferredDuringSchedulingIgnoredDuringExecution`) – prefer matching node, but fallback is allowed.
- Supports expression-based rules like `In`, `NotIn`, `Exists`, etc.

**Example:**

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

---

### 🔁 Summary:

| Feature         | nodeSelector            | nodeAffinity                              |
|----------------|-------------------------|-------------------------------------------|
| Rule Type      | Equality only           | Expression-based (`In`, `NotIn`, `Exists`)|
| Soft Affinity  | ❌ Not supported         | ✅ Supported                               |
| Hard Affinity  | ✅ Always                | ✅ (`requiredDuring...`)                  |
| Flexibility    | Low                     | High                                      |

💡 Use `nodeSelector` for quick, simple scheduling rules.  
💡 Use `nodeAffinity` when you need more control or fallback options.

### ❓ Question: What is a Container Runtime in Kubernetes?

### ✅ Answer:

A **Container Runtime** is the low-level component responsible for **running containers** on a Kubernetes node. It is a **core part of the Kubernetes data plane**, working closely with the kubelet to bring up containers as defined in pod manifests.

### 🔹 How It Fits Into Pod Lifecycle:

1. When the **API Server** receives a request to create a pod, it passes the scheduling task to the **Scheduler**.
2. The Scheduler picks a suitable node and the **API Server notifies the kubelet** running on that node.
3. The **kubelet** then uses the **Container Runtime** to:
   - **Pull the container image** from the container registry.
   - **Create necessary Linux primitives** like cgroups and namespaces.
   - **Start the container process** inside the isolated environment.

### 🔹 Common Container Runtimes:

- [containerd](w) (most popular today)
- [CRI-O](w)
- [Docker](w) (deprecated for Kubernetes after v1.20+)

💡 Think of the container runtime like the **Java Runtime Environment (JRE)** — just as Java code needs a JRE to run, **containers need a container runtime** to run.

Kubernetes interacts with the container runtime via the **Container Runtime Interface (CRI)**.

### ❓ What is Horizontal Pod Autoscaler (HPA)?

---

### ✅ Answer  
HPA is used to **automatically scale the number of pods** in a Deployment (or ReplicaSet) based on CPU/memory usage or custom metrics.

- If traffic or resource usage increases, HPA increases the **pod count**.  
- If load decreases, it **scales pods down**.  

💡 HPA scales **pods**, not nodes. Node scaling is handled by the **Cluster Autoscaler**.

---

### ❓ What are Liveness and Readiness Probes?

---

### ✅ Answer  
- **Liveness Probe** checks if the app is **alive** (running correctly). If it fails, Kubelet **restarts the container**.  
- **Readiness Probe** checks if the app is **ready to serve traffic**. If it fails, traffic is **not routed** to the pod.

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

---

### ❓ What are Volumes and PersistentVolumeClaims (PVCs) in Kubernetes?

---

### ✅ Answer  
- A **Volume** gives a pod access to storage beyond the container’s lifecycle.  
- A **PVC** is a request for storage by a user.  
- A **PV (PersistentVolume)** is the actual storage that backs the claim.

💡 You create a PVC → Kubernetes binds it to a matching PV → Pod mounts the PVC.

---

### ❓ What is Helm templating and how do you override values?

---

### ✅ Answer  
Helm uses templating to manage Kubernetes manifests dynamically.  
Templates allow values to be injected at runtime via `values.yaml` or `--set` flags.

You can override default values using:

```bash
helm install my-app ./chart -f custom-values.yaml
```

Or:

```bash
helm install my-app ./chart --set image.tag=v2
```

💡 This makes Helm ideal for managing multiple environments (dev, staging, prod).

---

### ❓ What’s the difference between Secrets and ConfigMaps?

---

### ✅ Answer  
- **Secrets** store sensitive data (like passwords, API keys) in Base64 encoded form.  
- **ConfigMaps** store non-sensitive configuration (like environment variables, ports).

💡 Both can be used as env variables or volume mounts inside pods.

---

### ❓ What are ServiceAccounts, Roles, and RBAC?

---

### ✅ Answer  
- **ServiceAccount**: Used by pods to interact with the Kubernetes API (like IAM roles for AWS resources).  
- **Roles/ClusterRoles**: Define what actions can be performed.  
- **RoleBindings/ClusterRoleBindings**: Define who can perform those actions.

💡 RBAC = Role-Based Access Control → ensures only the right users or services get access.

---

### ❓ How do you monitor the health of the Kubelet?

---

### ✅ Answer  
- Use `kubectl get nodes` to check status (`Ready`, `NotReady`).  
- Use `kubectl describe node <node-name>` for detailed kubelet logs.  
- Logs can also be checked via systemd:

```bash
journalctl -u kubelet
```

💡 Monitoring tools like Prometheus + Grafana can also help visualize kubelet health.

---

### ❓ What are Jobs and CronJobs in Kubernetes?

---

### ✅ Answer  
- A **Job** runs a task once until successful (e.g., database migration).  
- A **CronJob** runs on a schedule (like crontab), e.g., cleanup every night.

```yaml
schedule: "0 2 * * *"
```

💡 Great for periodic tasks, backups, or reporting jobs.

