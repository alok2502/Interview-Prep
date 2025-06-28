### Q: What is DNS and why is it important?

DNS (Domain Name System) is like the **phonebook of the internet**. Every website is hosted on a server that has an IP address (e.g., `142.250.195.46`), but remembering IPs for every site would be difficult.

Instead, we use **human-friendly domain names** like `google.com`, `amazon.in`, etc. DNS translates these domain names into their corresponding IP addresses behind the scenes.

Without DNS, users would have to type IP addresses to access websites, which is impractical. That’s why DNS is a core part of how the internet works — it allows us to use names instead of numbers.

### Q: Explain the complete flow of a request from client to server.

We can explain this using the OSI model, layer by layer, and map it to what actually happens when a client sends a request to a server.

1. **Application Layer (Layer 7)**  
   The user initiates a request — for example, by typing `www.example.com` into a browser. The browser uses protocols like HTTP or HTTPS to prepare the request.

2. **Presentation Layer (Layer 6)**  
   If encryption is used (e.g., HTTPS), data is encrypted here using TLS/SSL to ensure secure communication.

3. **Session Layer (Layer 5)**  
   This layer establishes, manages, and terminates sessions between the client and server — for instance, managing cookies or session tokens.

4. **Transport Layer (Layer 4)**  
   TCP or UDP handles reliable delivery. For HTTP/HTTPS, TCP is used. A 3-way handshake (SYN → SYN-ACK → ACK) happens to establish a connection.

5. **Network Layer (Layer 3)**  
   The IP protocol routes the data from the client to the server. DNS resolution also happens here to convert the domain name to an IP address.

6. **Data Link Layer (Layer 2)**  
   Converts the IP packet into frames and determines the MAC address for the next hop (router, gateway, etc.) using ARP.

7. **Physical Layer (Layer 1)**  
   Data is transmitted over the physical medium — cables, Wi-Fi, etc., in the form of electrical or optical signals.

#### Summary:
- The client request flows top-down through the OSI layers.
- It reaches the server, which processes it and sends back a response following the same path in reverse.
- Real-world components involved include DNS servers, load balancers, routers, switches, and the actual application server.

This layered breakdown helps isolate and troubleshoot issues in any part of the communication.

### Q: What is the difference between a Forward Proxy and a Reverse Proxy?

**Forward Proxy** and **Reverse Proxy** are both intermediaries, but they serve different roles in a network:

#### 🔁 Forward Proxy:
- Sits **in front of the client**.
- Used to control or filter outbound traffic from the client to the internet.
- Common in **corporate networks**, schools, or firewalled environments.
- Example: In an office, access to `youtube.com` is blocked — the forward proxy filters that request.
- It hides the **client's identity** from the server.

#### 🔄 Reverse Proxy:
- Sits **in front of the server**.
- Handles incoming client requests and routes them to the appropriate backend server or service.
- Commonly used for **load balancing, SSL termination, caching, and security**.
- Example: Nginx or HAProxy routes `/app`, `/home`, `/contact` to different microservices.
- It hides the **server's details** from the client.

#### Summary:
- **Forward proxy** protects and controls **clients**.
- **Reverse proxy** protects and optimizes **servers**.

  ### Q: A user reports slowness on a server. How will you respond?

1. **Login to the server** and check system resource usage:
   - Use commands like:
     ```bash
     top
     htop
     ```
   - Check for high CPU or memory utilization.

2. **Identify high resource-consuming processes**:
   - If the process is **non-critical**, I’ll kill it:
     ```bash
     kill -9 <PID>
     ```
   - If the process is **critical**, I’ll deprioritize it using:
     ```bash
     renice +10 <PID>
     ```

3. **Check for network latency or packet loss**:
   - Use:
     ```bash
     ping <destination>
     traceroute <destination>
     ```
   - If high latency is observed, I’ll recommend using a **CDN** or hosting the server **closer to the users** (geolocation-based optimization).

4. **Check application logs**:
   - Inspect `/var/log`, app-specific logs, and system logs to identify potential bottlenecks or errors.

5. **Communicate**:
   - If an issue is identified, I’ll inform the team and begin remediation.
   - If no issue is found at the system/network level, I’ll escalate to the app team for further investigation.

### ❓ Q: From a VM, I'm able to `curl` to an IP, but not access the same app via the domain name. What could be the issue?

This is most likely a **DNS resolution issue** — the system can reach the IP directly, but cannot resolve the domain name to that IP.

### 🧪 Steps to Troubleshoot

#### 1. ✅ Confirm the issue:
```bash
curl <IP>              # works
curl http://domain.com # fails
```
#### 2. 🔍 Check current DNS settings:

Inspect the DNS server being used:
```bash
cat /etc/resolv.conf
```
#### 3. 🧠 Test DNS resolution manually:
```bash
nslookup domain.com
dig domain.com
```
> If these fail, it confirms a DNS issue.
#### 4. 🛠️ Fix DNS temporarily:

Edit `/etc/resolv.conf` and set a reliable public DNS:
```bash
nameserver 8.8.8.8  # Google's DNS
```

> ⚠️ Note: This is a temporary change — some systems regenerate this file on reboot or network restart.

#### 5. 🔧 Permanent fix (optional):

Update DNS settings via:
- `/etc/systemd/resolved.conf`  
- Or your OS/network manager configuration

### ❓ Q: A user reports a 502 Bad Gateway error. What could be the issue and how will you approach it?

A **502 Bad Gateway** error means the **reverse proxy** (e.g., Nginx, HAProxy, Load Balancer) received an invalid or no response from the **upstream/backend server**.

This indicates the **client is fine**, and the issue lies between the **proxy and backend** services.

---

### 🔍 Possible Causes

1. **Backend server is down** or unreachable (crashed, stopped, or network issues).
2. **Backend service is running on the wrong port** or configured incorrectly.
3. **Timeouts** — backend is too slow to respond.
4. **Misconfiguration** in the reverse proxy (e.g., wrong upstream host/port).
5. **Health checks failing** in a load balancer (common in cloud setups).
6. **Firewall/Security Group rules** blocking internal traffic between components.


### 🛠️ How I'd Troubleshoot

#### 1. Check if the backend service is active:
```bash
systemctl status <backend_service>
```

#### 2. Test connectivity between proxy and backend:
```bash
curl http://localhost:<port>
```

#### 3. Review reverse proxy configuration:
Look into config files like:
- `/etc/nginx/nginx.conf`
- `/etc/nginx/sites-enabled/your-site.conf`

Ensure `proxy_pass` or `upstream` is set to the correct target.

#### 4. Inspect logs for clues:

**Nginx error logs:**
```bash
tail -n 100 /var/log/nginx/error.log
```

**Application/backend logs:**  
Check for crashes, timeouts, or resource bottlenecks.

#### 5. Verify firewall or security rules:
Ensure internal traffic is allowed between the proxy and backend:
- Check `iptables`, `ufw`, or cloud-level security groups

### Q: What is the difference between 127.0.0.1 and 0.0.0.0?

- **127.0.0.1** is the **loopback address**.  
  It is used when a machine wants to communicate with **itself**.  
  - Example: `curl http://127.0.0.1:8080` tests an app running locally.
  - Only accessible from the same machine.

- **0.0.0.0** is a **wildcard address**.  
  It tells the system to **listen on all IPv4 interfaces** on the machine.  
  - Example: A web server bound to `0.0.0.0:80` will accept requests from any network interface (localhost, private IP, public IP).

#### Summary:
- `127.0.0.1` = listen to **self only**
- `0.0.0.0` = listen to **everyone**

### Q: What is the difference between `localhost` and `127.0.0.1`?

- **127.0.0.1** is the **loopback IP address**, part of the reserved IPv4 loopback range.
  - It's a numeric address used by the system to refer to itself.
  - Example: `ping 127.0.0.1`

- **localhost** is a **hostname**.
  - It typically resolves to `127.0.0.1` via `/etc/hosts` or DNS.
  - It relies on name resolution (like DNS or local host file).
  - Example: `ping localhost`

#### Key Differences:
| Aspect        | 127.0.0.1        | localhost           |
|---------------|------------------|----------------------|
| Type          | IP Address        | Hostname             |
| Resolution    | Direct (numeric)  | Requires name resolution |
| Configurable? | No (fixed IP)     | Yes (can map to other IPs in `/etc/hosts`) |
| Usage         | Network testing, app configs | Often used in dev/testing environments |

#### Summary:
- Functionally, they behave the same **in most cases**.
- But `localhost` relies on name resolution, while `127.0.0.1` is a direct IP — which makes `127.0.0.1` slightly more reliable when troubleshooting name resolution problems.

### Q: What is the difference between a public subnet and a private subnet?

#### 🔹 Public Subnet:
- A **public subnet** is a subnet whose resources **can communicate directly with the internet**.
- This is possible because it is associated with a **Route Table** that has a route to an **Internet Gateway**.
- Typically used to host:
  - Web servers
  - Load balancers
  - NAT Gateways

#### 🔒 Private Subnet:
- A **private subnet** is a subnet that **does not have direct access to the internet**.
- There is **no route to an Internet Gateway**.
- More secure — ideal for sensitive/internal components.
- Typically used to host:
  - Databases
  - Application backend servers
  - Internal microservices

#### Example Use Case:
- Public Subnet: Load balancer or frontend app (accessible from internet)
- Private Subnet: Database or internal APIs (not exposed to public)

#### Summary:
- **Public subnet** → internet-facing (via Internet Gateway)
- **Private subnet** → isolated, secure, internal-facing

### Q: You accidentally created a private subnet instead of a public one. How will you fix it?

You don’t need to delete or recreate the subnet — just update its route table to give it internet access.

#### ✅ Steps to convert a private subnet into a public subnet:

1. **Attach an Internet Gateway (IGW) to the VPC** (if not already attached):
   - Go to VPC → Internet Gateway → Create & Attach to your VPC.

2. **Create or modify a Route Table**:
   - Go to Route Tables → Select the one associated with your subnet (or create a new one).
   - Add a route:
     ```text
     Destination: 0.0.0.0/0
     Target: Internet Gateway (your IGW ID)
     ```

3. **Associate the Route Table with the Subnet**:
   - In the subnet settings, associate it with the updated route table.

4. **(Optional) Modify Network ACLs and Security Groups**:
   - Ensure that **inbound/outbound rules** allow traffic (e.g., allow TCP port 80/443/22 as needed).

5. **Assign a Public IP to instances**:
   - Either enable **Auto-Assign Public IP** on the subnet level or manually attach an Elastic IP to instances.

#### Summary:
To make a subnet public, it must:
- Be associated with a route table that routes `0.0.0.0/0` traffic to an **Internet Gateway**.
- Have instances with **public IPs** and open **security groups**.




