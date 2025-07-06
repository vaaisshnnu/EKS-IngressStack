# 🚀 Project: EKS IngressStack - Deploying Dockerized Web Apps on AWS EKS with Ingress Controller

> **Author**: Buildby-GangaVaishnuReddy

A complete end-to-end setup of deploying multiple static applications using Docker, Kubernetes, AWS EKS, and Ingress Controller. This README documents all steps, commands, and directory structure exactly as implemented.

---

## 🗂️ Directory Structure

```bash
/home/ubuntu/
├── aws/
├── awscliv2.zip
├── install-awscli.sh
├── install-ekscli.sh
├── install-kubectl.sh
├── installdocker.sh
├── k8s/
│   ├── app1-deploy.yaml
│   ├── app2-deploy.yaml
│   ├── app3-deploy.yaml
│   └── ingress.yaml
└── my_app/
    ├── app1/
    │   ├── index.html
    │   └── Dockerfile
    ├── app2/
    │   ├── index.html
    │   └── Dockerfile
    └── app3/
        ├── index.html
        └── Dockerfile
```

---

## 🛠️ Step-by-Step Setup

### 🔧 1. Update and Upgrade Packages

```bash
sudo su -
sudo apt update -y       # Updates package list
sudo apt upgrade -y      # Installs available upgrades
```

### 📦 2. AWS CLI Installation Script

```bash
vi install-awscli.sh
```

```bash
#!/bin/bash
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"   # Downloads AWS CLI zip
unzip awscliv2.zip              # Extracts it
sudo ./aws/install              # Installs AWS CLI
aws --version                   # Verifies installation
```

### 📦 3. EKSCTL Installation Script

```bash
vi install-ekscli.sh
```

```bash
#!/bin/bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp  # Downloads eksctl
sudo mv /tmp/eksctl /usr/local/bin      # Moves it to PATH
eksctl version                         # Verifies installation
```

### 📦 4. KUBECTL Installation Script

```bash
vi install-kubectl.sh
```

```bash
#!/bin/bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl  # Downloads kubectl
chmod +x ./kubectl              # Makes it executable
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client  # Verifies installation
```

### 📦 5. Docker Installation Script

```bash
vi installdocker.sh
```

```bash
#!/bin/bash
sudo apt update -y
sudo apt upgrade -y
sudo apt install -y docker.io       # Installs Docker
sudo systemctl start docker         # Starts Docker service
sudo systemctl enable docker        # Enables Docker to start on boot
sudo chmod 666 /var/run/docker.sock # Sets Docker socket permissions
```

### ✅ 6. Make Scripts Executable

```bash
chmod +x installdocker.sh install-kubectl.sh install-awscli.sh install-ekscli.sh
```

---

## 🔐 7. AWS IAM Setup

* Create IAM user
* Attach the following policies:

  * `AdministratorAccess`
  * `AmazonEC2FullAccess`
  * `AmazonVPCFullAccess`
  * `AWSCloudFormationFullAccess`
  * `IAMFullAccess`
* Save Access Key & Secret Key or download the `.csv`

### ⚙️ 8. AWS Configure on EC2

```bash
aws configure
# Enter access key, secret, region (e.g., ap-south-1), output format (json)
```

---

## ☸️ 9. Create EKS Cluster

```bash
eksctl create cluster --name eks-cluster --region ap-south-1 --node-type t2.medium --zones ap-south-1a,ap-south-1b
```

---

## 📁 10. Application Directory & Files

```bash
mkdir my_app
cd my_app
mkdir app1 app2 app3
```

Each app directory contains:

* `index.html` – A creative HTML page
* Code for App1- inde.html[https://github.com/vaaisshnnu/EKS-IngressStack/blob/main/APP1%20-%20Index.html]
* `Dockerfile` – Builds image using nginx base

### Example Dockerfile (Same for all apps):

```Dockerfile
FROM nginx:alpine
COPY ./index.html /usr/share/nginx/html/index.html
```

---

## 🐳 11. Build and Push Docker Images

```bash
docker build -t app1-image /home/ubuntu/my_app/app1/.
docker login -u <your_username>
docker tag app1-image:latest vaaisshnnu/app1-image:latest
docker push vaaisshnnu/app1-image:latest

# Repeat for App2 & App3
docker build -t app2-image /home/ubuntu/my_app/app2/.
docker tag app2-image:latest vaaisshnnu/app2-image
\
docker push vaaisshnnu/app2-image:latest

docker build -t app3-image /home/ubuntu/my_app/app3/.
docker tag app3-image:latest vaaisshnnu/app3-image
\
docker push vaaisshnnu/app3-image:latest
```

---

## ☸️ 12. Create Kubernetes YAMLs

```bash
mkdir k8s
cd k8s
```

* `app1-deploy.yaml`, `app2-deploy.yaml`, `app3-deploy.yaml` — contain Deployment and NodePort Service for each app
* `ingress.yaml` — routes traffic to respective services

### Apply the manifests:

```bash
kubectl apply -f app1-deploy.yaml
kubectl apply -f app2-deploy.yaml
kubectl apply -f app3-deploy.yaml
```

### Verify:

```bash
kubectl get svc
kubectl get deploy
kubectl get pods
```

> ⚠️ Ensure Security Group allows TCP 30000–32767 for NodePort access

---

## 🌐 13. Install Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml
```

Verify:

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

Get External IP from `EXTERNAL-IP` field for ingress

---

## 🔁 14. Create and Apply Ingress Resource

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: <your-elb-url>
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app1-svc
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app2-svc
                port:
                  number: 80
          - path: /app3
            pathType: Prefix
            backend:
              service:
                name: app3-svc
                port:
                  number: 80
```

```bash
kubectl apply -f ingress.yaml
```

---

## ✅ 15. Access Apps

Replace `<elb-host>` with LoadBalancer DNS (from Ingress svc):

```url
http://<elb-host>/app1
http://<elb-host>/app2
http://<elb-host>/app3
```

Ensure EC2 Security Group allows ports 80 & 443 (HTTP/HTTPS)

---

## 🎯 Project Outcome

* 3 Static apps hosted on AWS EKS
* Dockerized and pushed to Docker Hub
* Routed via Ingress Controller
* Real-world Kubernetes hands-on with custom branding

---

## 📸 Architecture Diagram (Provided separately below)

---

> Happy Cloud Building! 🚀
>
> **Buildby-GangaVaishnuReddy**
