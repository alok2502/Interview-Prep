### Q: What are the 10 day-to-day Linux commands that you use?

As a DevOps engineer, I use a variety of Linux commands daily. Here are the top 10 I rely on the most:

1. **pwd** – Prints the current working directory. Helpful when navigating deep folder structures.
2. **ls** – Lists files and directories. I often use options like `-a`, `-l`, and `-ltr` for more details or sorting.
3. **cd** – Used to navigate between directories.
4. **grep** – For searching patterns within files or output, especially useful in logs.
5. **chmod / chown** – Since we follow RBAC (Role-Based Access Control), I frequently adjust file permissions or ownership.
6. **systemctl** – I often use it to check the status of services like Docker, Jenkins, or Nginx (e.g., `systemctl status jenkins`).
7. **top / htop / ps** – For monitoring system performance, checking running processes, and CPU/memory usage.
8. **df -h** – Shows disk space usage in a human-readable format.
9. **free -m** – To check memory usage (RAM and swap).
10. **cat / less / tail / head** – For reading and viewing log files or configs quickly.

Other commonly used commands include `touch`, `mkdir`, and `rm` for creating and deleting files or directories.

### Q: Can you restore a lost PEM file?

No, the original `.pem` file (private key) cannot be recovered once lost. AWS does not store private keys, so if you didn’t back it up, it’s gone.

#### Follow-up: If not, how will you access that EC2 instance?

There are a few ways to regain access:

##### ✅ Method 1: Use EC2 Instance Connect (if enabled)
If the EC2 instance supports EC2 Instance Connect (e.g., Amazon Linux 2) and has:
- A public IP
- Port 22 open in the security group
- Proper IAM permissions

Then you can use **EC2 Connect from the AWS Console** to access the instance without a `.pem` file.

##### ✅ Method 2: Use Systems Manager Session Manager
If your EC2 instance:
- Has the **SSM agent installed**
- Is assigned an **IAM role with SSM permissions**
- Has internet access or a VPC SSM endpoint

Then you can connect via **Session Manager**, even without SSH access.

##### ✅ Method 3: Replace the key manually (fallback method)
If the above methods are not available, you can:
1. Stop the EC2 instance.
2. Detach its root EBS volume.
3. Attach it to another EC2 instance you can access.
4. Modify the `authorized_keys` file to add a new public key.
5. Reattach the volume to the original instance and boot it.
6. SSH using the new private key.

This ensures access even without EC2 Connect or SSM preconfigured.

### Q: Let's say /var on one of your EC2 instances is 90% full. What steps will you take?

The `/var` directory often fills up due to logs. Here's how I'd handle it:

1. Check which subdirectory is consuming space:

    ```bash
    sudo du -sh /var/* | sort -h
    ```

2. If it's `/var/log`, inspect large files:

    ```bash
    sudo du -sh /var/log/*
    journalctl --disk-usage   # If using systemd
    ```

3. Take action:
    - Run log rotation:
        ```bash
        sudo logrotate -f /etc/logrotate.conf
        ```
    - Compress large old logs:
        ```bash
        sudo gzip /var/log/filename.log
        ```
    - Delete safe-to-remove logs only after review or backup.

4. Clean package cache:

    ```bash
    sudo apt clean    # or sudo yum clean all
    ```

5. If space is still tight, expand the EBS volume or move logs to a separate mounted disk.

### Q: On a production instance, slowness is observed due to high CPU utilization. What will you do?

In a production environment, performance issues need immediate attention to minimize downtime. Here's how I would respond:

1. **SSH into the instance** immediately.

2. **Check real-time CPU usage** using:

    ```bash
    top    # or htop, if available
    ```

3. **Identify high CPU-consuming processes**. If I need more detailed info, I'd use:

    ```bash
    ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
    ```

4. **Action based on process criticality**:
    - If non-critical:
        ```bash
        kill -9 <PID>
        ```
    - If critical but resource-heavy:
        ```bash
        renice +10 <PID>
        ```

5. Optionally, analyze system stats:
    ```bash
    iostat, vmstat, sar
    ```

6. After stabilization, notify the team and collect logs for root cause analysis.

---

### Q: Do you need more time to investigate this?

No, since it’s a production issue, we can't afford downtime. I’ll act immediately to stabilize the system.  
For non-prod environments, I would take more time to investigate and share logs with the dev team.

### Q: An application deployed on Nginx is returning "Connection Refused". How will you fix it?

1. **SSH into the server**.

2. **Check if Nginx is running**:
    ```bash
    sudo systemctl status nginx
    ```
    - If it's not running, start it:
      ```bash
      sudo systemctl start nginx
      ```
    - Then check why it stopped:
      ```bash
      sudo journalctl -u nginx
      ```

3. **Verify if the application is actually listening**:
    ```bash
    sudo netstat -tuln | grep 80
    ```
    or
    ```bash
    ss -tuln | grep 80
    ```
    - Confirm that either Nginx or your upstream app is listening on the correct port (80 or 443).

4. **Check Nginx configuration**:
    ```bash
    sudo nginx -t
    ```
    - This tests for syntax errors in `nginx.conf`.

5. **Check firewall rules**:
    - For EC2: Make sure port 80 (and 443 if HTTPS) is open in the **Security Group** inbound rules.
    - Also check `ufw` or `iptables` on the OS level:
      ```bash
      sudo ufw status
      sudo iptables -L
      ```

6. **Restart Nginx after config changes**:
    ```bash
    sudo systemctl restart nginx
    ```

7. If the application is running behind Nginx (like on port 3000), make sure **proxy_pass** in the config is pointing to the correct internal port.

This approach covers service status, logs, port binding, firewall rules, and app linkage.

### Q: SSH to an instance has stopped working. How will you troubleshoot it?

If SSH access to an instance fails, I’ll go through the following checklist:

1. **Verify SSH command syntax and details**:
    - Ensure the correct IP address is being used.
    - Confirm the correct username (e.g., `ec2-user`, `ubuntu`, etc.).
    - Make sure the correct `.pem` file is specified.

2. **Check if the IP address has changed**:
    - For cloud providers like AWS, public IPs may change after stopping/starting an instance (unless using Elastic IP).
    - Update SSH command if needed.

3. **Check PEM file permissions**:
    - Ensure the key file has restricted permissions:
      ```bash
      chmod 400 my-key.pem
      ```

4. **Ping the instance**:
    - Use `ping <ip>` to see if the instance is reachable at all.
    - If ping fails, it could be a networking/VPC issue.

5. **Check firewall or security group settings**:
    - On-prem: ensure OS firewall (`ufw`/`firewalld`) allows port 22 or is disabled.
      ```bash
      sudo ufw status
      sudo systemctl stop firewalld
      ```
    - On AWS: verify the EC2 **Security Group** has port 22 open in **inbound rules**.

6. **Ensure the SSH service is running** (if you have alternate access):
    ```bash
    sudo systemctl status ssh
    ```

7. **Fallback options (cloud only)**:
    - For AWS EC2, use **EC2 Instance Connect** or **SSM Session Manager** (if configured) to regain access and fix SSH issues from inside.

Once access is restored, I’d check `/var/log/auth.log` or `/var/log/secure` for deeper SSH error investigation.

### Q: How will you delete `.log` files in `/var/log` that are older than 7 days?

You can use the following `find` command:

```bash
find /var/log -type f -name "*.log" -mtime +7 -exec rm {} \;
```
This finds all .log files older than 7 days and deletes them.

The \; at the end properly terminates the -exec command.

### Q: Using `sed`, how will you delete the first and last lines of a file?

You can use the following command:

```bash
sed -i '1d;$d' file.txt
```



