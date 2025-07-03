# 🐳 Docker Interview Questions & Answers

---

### ❓ Question 1  
You have just built a Docker image using the command `docker run`, but the container exits immediately. What can be the reason and how will you troubleshoot it?

### ✅ Answer  
Docker containers are designed to run a **single main process**. If the command provided via `CMD` or `ENTRYPOINT` completes quickly (e.g., `echo "hello world"`) or **fails due to an error**, the container will exit immediately.

#### 🔍 Troubleshooting  
- Use `docker logs <container-id>` to check for errors.  
- For debugging:
  ```bash
  docker run -it <image-name> /bin/bash
  ```

---

### ❓ Question 2  
What is the purpose of the `EXPOSE` keyword in a Dockerfile?

### ✅ Answer  
`EXPOSE` is **metadata**, not a firewall rule. It tells Docker which port your container expects to listen on, but doesn’t publish it externally.

#### 🧠 Real Use  
Use `-p` to expose to the host:

```bash
docker run -p 8080:8080 myapp
```

In Docker Compose, `EXPOSE` helps containers communicate internally.

---

### ❓ Question 3  
The port is not accessible even after using port mapping. What could be the reason?

### ✅ Answer  
#### 🔍 Possible Causes  
1. Incorrect port mapping  
2. Host port already in use  
3. Firewall blocking traffic  
4. App crash or not binding correctly  
5. Container bound to wrong IP

#### 🛠️ Commands  
```bash
netstat -tuln         # Check port status
docker logs <id>      # Check container output
docker inspect <id>   # Check bind IP/port
```

---

### ❓ Question 4  
Data is lost when a container stops. How do you prevent this?

### ✅ Answer  
Use **Docker volumes** to persist data outside the container.

#### 🔧 Example  
```bash
docker volume create mydata
docker run -v mydata:/app/data myimage
```

---

### ❓ Question 5  
You rebuilt the image but changes didn’t reflect. What’s wrong?

### ✅ Answer  
Probably Docker **cached layers** and reused them.

#### 🔍 Fixes  
```bash
docker build --no-cache -t myapp .
```

Ensure `COPY`/`ADD` comes after dependency install in your Dockerfile.  
Also make sure you're running the updated image:

```bash
docker ps -a
docker rm -f <old-id>
docker run myapp
```

---

### ❓ Question 6  
App crashes with "permission denied" inside the container. Why?

### ✅ Answer  
Likely due to **user mismatch** or missing permissions.

#### 🛠️ Fix  
```dockerfile
USER myuser
RUN chown -R myuser:myuser /app
RUN chmod +x start.sh
```

---

### ❓ Question 7  
Docker host is out of disk space. How do you clean it?

### ✅ Answer  
#### 🧹 Commands  
```bash
docker system df
docker system prune -a
docker volume prune
```

⚠️ *Use with care in production.*

---

### ❓ Question 8  
How do you debug a live container?

### ✅ Answer  
Use `docker exec` to enter the container.

```bash
docker exec -it <id> /bin/bash
# Or fallback:
docker exec -it <id> /bin/sh
```

Check logs, configs, permissions, and env variables from inside.

---

### ❓ Question 9  
Which container registry do you use?

### ✅ Answer  
We use **Amazon ECR** due to AWS integration, secure IAM access, and performance within AWS infra.

---

### ❓ Question 10  
Explain the difference between `CMD` and `ENTRYPOINT`.

### ✅ Answer  
- `CMD`: Default args, overrideable.
- `ENTRYPOINT`: Base command, args get appended.

#### 🧪 Example  
```dockerfile
ENTRYPOINT ["npm"]
CMD ["start"]
```

```bash
docker run myapp test  # Runs: npm test
```

---

### ❓ Question 11  
What Docker commands do you use daily?

### ✅ Answer  
```bash
docker build -t myapp .
docker run -d -p 8080:80 myapp
docker push myrepo/myapp:latest
docker stop <id>
docker ps
docker images
docker rm -f <id>          # Force remove stuck containers
```

These are essential for building, running, pushing, and managing containers.

---
