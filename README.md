# 🚀 Directus CI/CD Deployment on AWS (Beginner Friendly)

This repository demonstrates a **complete end‑to‑end CI/CD pipeline** to deploy **Directus** on an **AWS EC2 instance** using **GitHub, Jenkins, Docker, Docker Compose, and NGINX**.


---

## 🧱 Tech Stack

* **Cloud**: AWS EC2 (Ubuntu)
* **CI/CD**: Jenkins
* **Containerization**: Docker & Docker Compose
* **Database**: PostgreSQL
* **Backend**: Directus
* **Reverse Proxy**: NGINX
* **Source Control**: GitHub

---

## 📁 Project Structure

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

## ⚙️ Prerequisites

* AWS EC2 Ubuntu instance
* Ports opened in Security Group:

  * 22 (SSH)
  * 80 (HTTP)
  * 8080 (Jenkins)
* GitHub repository
* Jenkins installed on EC2

---

## 🔑 Step 1: Install Required Packages on EC2

```bash
sudo apt update -y
sudo apt install -y git docker.io docker-compose nginx
```

Enable and start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins
```

Restart services:

```bash
sudo systemctl restart docker
sudo systemctl restart jenkins
```

---

## 🐳 Step 2: Docker Compose Configuration

**docker-compose.yml**

```yaml
version: '3'

services:
  database:
    image: postgres:15
    container_name: directus-db
    environment:
      POSTGRES_USER: directus
      POSTGRES_PASSWORD: directus
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
      DB_PASSWORD: directus
    depends_on:
      - database

volumes:
  db_data:
```

---

## 🔐 Step 3: Environment Variables (.env)

```env
KEY=mysecretkey
SECRET=mysecret
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=Admin@123
```

---

## 🔁 Step 4: Jenkins Pipeline

**Jenkinsfile**

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout Code') {
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

## 🌐 Step 5: NGINX Reverse Proxy

**nginx/default.conf**

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

Enable config:

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /home/ubuntu/directus-prod/nginx/default.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---

## ▶️ Step 6: Jenkins Job Configuration

* **Pipeline type**: Pipeline script from SCM
* **SCM**: Git
* **Branch**: `*/main`
* **Script Path**: `Jenkinsfile`

---

## ✅ Verification

Check containers:

```bash
docker ps
```

Open browser:

```
http://<EC2_PUBLIC_IP>
```

Directus login page should load.

---

## 🧪 Troubleshooting

### Branch Error

```
fatal: couldn't find remote ref refs/heads/master
```

✔ Fix: Change branch to `main` in Jenkins.

### Directory Error

```
cd: can't cd to directory
```

✔ Fix: Use `$WORKSPACE` in Jenkinsfile.

---

## 🚀 Future Enhancements

* HTTPS using Let's Encrypt
* GitHub Webhooks for auto‑deploy
* AWS RDS instead of container DB
* Secrets via AWS SSM
* Monitoring with Prometheus

---

## 👨‍💻 Author

**Deepak Rathour**
DevOps Fresher | AWS | Docker | Jenkins

---

⭐ If this helped you, star the repository!
