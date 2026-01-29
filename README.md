# 🚀 Deploying Directus on AWS with Jenkins and Docker

This repository shows how to set up a **full CI/CD pipeline** for deploying **Directus** on an **AWS EC2 instance**, using **GitHub for source control, Jenkins for automation, Docker for containerization, and NGINX as a reverse proxy**.

The guide is written step-by-step, so you can follow along even if you’re new to DevOps.

---

## 🛠 Tech Stack

* **Cloud:** AWS EC2 (Ubuntu 24.04)
* **CI/CD:** Jenkins
* **Containerization:** Docker & Docker Compose
* **Database:** PostgreSQL
* **Backend:** Directus
* **Web Server:** NGINX
* **Version Control:** GitHub

---

## 📂 Project Layout

```
directus-devops--prjt/
│
├── docker-compose.yml
├── .env
├── Jenkinsfile
├── nginx/
│   └── default.conf
└── README.md
```

---

## ⚡ Prerequisites

Before starting, make sure you have:

* An AWS EC2 instance (Ubuntu 24.04)
* Open ports in your security group: 22 (SSH), 80 (HTTP), 8080 (Jenkins)
* A GitHub repository for the project
* Access to the SSH key for the EC2 instance

---

## 1️⃣ Connect to Your EC2 Instance

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@13.233.199.162
```

---

## 2️⃣ Update and Upgrade System Packages

```bash
sudo apt update -y
sudo apt upgrade -y
```

---

## 3️⃣ Install Java (Required for Jenkins)

```bash
sudo apt install -y openjdk-21-jre fontconfig
java -version
```

---

## 4️⃣ Install Jenkins

```bash
sudo mkdir -p /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update -y
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

Access Jenkins: `http://13.233.199.162:8080`

Unlock Jenkins:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Install suggested plugins: Git, Pipeline, Docker Pipeline, GitHub Integration, GitHub Branch Source, Workspace Cleanup.

---

## 5️⃣ Install Docker & Docker Compose

```bash
sudo apt install -y docker.io docker-compose
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
newgrp docker

docker --version
docker-compose --version
```

---

## 6️⃣ Setup Project Folder

```bash
cd ~
mkdir directus-prod
cd directus-prod
```

---

## 7️⃣ Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
version: '3'

services:
  database:
    image: postgres:15
    container_name: directus-db
    environment:
      POSTGRES_USER: directus
      POSTGRES_PASSWORD: Directus@123
      POSTGRES_DB: directus
    volumes:
      - db_data:/var/lib/postgresql/data

  directus:
    image: directus/directus:latest
    container_name: directus-app
    ports:
      - "8055:8055"
    environment:
      KEY: mysecretkey
      SECRET: mysecret
      DB_CLIENT: pg
      DB_HOST: database
      DB_PORT: 5432
      DB_DATABASE: directus
      DB_USER: directus
      DB_PASSWORD: Directus@123
    depends_on:
      - database

volumes:
  db_data:
```

---

## 8️⃣ Environment Variables

Create `.env`:

```env
KEY=mysecretkey
SECRET=mysecret
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=Admin@123
```

---

## 9️⃣ Manual Test

```bash
docker-compose up -d
docker ps
```

Visit: `http://13.233.199.162:8055`

Stop containers:

```bash
docker-compose down
```

---

## 🔁 10️⃣ GitHub Setup

```bash
git init
git branch -M main
git remote add origin https://github.com/DeepakkumarRathour/directus-devops--prjt.git
git add .
git commit -m "Production-ready Directus deployment"
git push -u origin main
```

---

## 11️⃣ Jenkins Pipeline

Create `Jenkinsfile`:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DeepakkumarRathour/directus-devops--prjt.git'
            }
        }

        stage('Deploy Directus') {
            steps {
                sh '''
                cd $WORKSPACE
                docker-compose down || true
                docker-compose up -d --build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh 'curl -I http://localhost:8055'
            }
        }
    }
}
```

---

## 12️⃣ NGINX Reverse Proxy

Create `nginx/default.conf`:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://localhost:8055;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable and restart NGINX:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /home/ubuntu/directus-prod/nginx/default.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---

## 13️⃣ Jenkins Job Configuration

* Type: **Pipeline**
* Pipeline script from SCM
* SCM: Git, branch: `*/main`
* Script Path: `Jenkinsfile`
* Save → Build Now

---

## 14️⃣ Verification

```bash
docker ps
```

Visit: `http://13.233.199.162`

Directus login should appear.

---

## 🌟 Future Enhancements

* HTTPS with Let’s Encrypt
* GitHub Webhooks for auto-deploy
* AWS RDS for database
* Secrets via AWS SSM
* Monitoring with Prometheus/Grafana

---

## 👤 Author

**Deepak Rathour** — DevOps Enthusiast | AWS | Docker | Jenkins

---

⭐ If this guide helped, please star the repository!
