All the troubleshooting steps you encountered with your EC2 + Docker + ECR setup**. This can go straight into your **GitHub project README** or your **resume/portfolio**. I’ll structure it so it’s professional, clear, and shows your DevOps problem-solving skills.

---

# **Hybrid Cloud DevOps Project – Troubleshooting & Solutions**

### **1️⃣ Issue: Docker Build Fails Locally**

**Error:**

```text
unable to prepare context: unable to evaluate symlinks in Dockerfile path: lstat /home/tummie/Dockerfile: no such file or directory
```

**Cause:**

* Docker could not find a `Dockerfile` in the current directory.

**Solution:**

1. Create a project folder:

```bash
mkdir hybrid-app && cd hybrid-app
```

2. Add a simple app:

```bash
mkdir app
echo "<h1>Hybrid Cloud DevOps Project</h1>" > app/index.html
```

3. Create Dockerfile:

```dockerfile
FROM nginx:latest
COPY app/ /usr/share/nginx/html/
EXPOSE 80
```

4. Build Docker image:

```bash
docker build -t hybrid-app:latest .
```

**Outcome:** Local Docker build works ✅

---

### **2️⃣ Issue: EC2 Cannot Pull Docker Image From ECR**

**Error:**

```text
no basic auth credentials
```

**Cause:**

* Docker on EC2 was not authenticated to AWS ECR.
* Your IAM user credentials were only configured on your **local machine**, not EC2.

**Solution Options:**

**Option A – Quick Lab Method: IAM User**

1. Install AWS CLI on EC2:

```bash
sudo apt update
sudo apt install awscli -y
```

2. Configure AWS CLI with IAM user credentials:

```bash
aws configure
# Enter IAM Access Key, Secret Key, region eu-west-1
```

3. Authenticate Docker to ECR:

```bash
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <ECR-URL>
```

4. Pull the image:

```bash
docker pull <ECR-URL>/hybrid-app:latest
```

---

**Option B – Enterprise Method: IAM Role (Recommended)**

1. Create IAM Role:

* Trusted entity: **AWS service → EC2**
* Policy: `AmazonEC2ContainerRegistryReadOnly`
* Name: `EC2-ECR-ReadOnly`

2. Attach role to EC2 instance via AWS Console
3. Verify role:

```bash
aws sts get-caller-identity
```

4. Authenticate Docker and pull image as in Option A

**Outcome:** EC2 can pull images securely, no credentials stored ✅

---

### **3️⃣ Issue: Application Not Loading in Browser**

**Symptoms:**

* `curl localhost` on EC2 shows app ✅
* Browser shows default Nginx page or cannot connect ❌

**Causes:**

1. EC2 **Security Group** not allowing inbound HTTP (port 80)
2. Host EC2 Nginx conflict with container Nginx

**Solutions:**

**Step 1 – Configure Security Group**

* EC2 → Security → Inbound Rules → Add:

| Type | Protocol | Port | Source    |
| ---- | -------- | ---- | --------- |
| HTTP | TCP      | 80   | 0.0.0.0/0 |

**Step 2 – Stop Host Nginx**

```bash
sudo systemctl stop nginx
sudo systemctl disable nginx
```

**Step 3 – Run Docker Container on EC2**

```bash
docker ps -q | xargs docker stop
docker ps -aq | xargs docker rm
docker run -d -p 80:80 <ECR-URL>/hybrid-app:latest
```

**Step 4 – Test**

* On EC2:

```bash
curl localhost
```

* On browser:

```
http://<EC2-Public-IP>
```

**Outcome:** App loads correctly ✅

---

### **4️⃣ Lessons Learned / DevOps Highlights**

| Topic                   | Takeaway                                                                                              |
| ----------------------- | ----------------------------------------------------------------------------------------------------- |
| Docker                  | Always verify Dockerfile location and context                                                         |
| ECR Authentication      | EC2 must authenticate separately; IAM Roles preferred                                                 |
| Security Groups         | Always allow inbound ports for services exposed publicly                                              |
| Host vs Container Nginx | Stop host service to avoid port conflicts                                                             |
| Troubleshooting         | Use `curl localhost`, `docker ps`, and `docker logs` to debug containerized apps                      |
| Resume Value            | Demonstrates hands-on hybrid cloud deployment, secure container registry access, and debugging skills |

---


**curl output****
<img width="720" height="491" alt="image" src="https://github.com/user-attachments/assets/37260c47-bfea-4b06-9b35-fd4a60487e25" />

  
** browser******
<img width="940" height="280" alt="image" src="https://github.com/user-attachments/assets/1dc14eb2-cba3-45bd-a31c-458234f1726a" />



