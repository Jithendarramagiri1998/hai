# 🚀 Jenkins CI/CD Setup (Master-Agent) with GitHub & HTML Deployment

This guide explains how to set up a Jenkins master-agent architecture using EC2 instances and GitHub for deploying a simple HTML application through a CI/CD pipeline.

---

## 📁 Architecture Overview

```
GitHub Repo → Jenkins Master (EC2)
                  |
             [SSH Connection]
                  ↓
           Jenkins Agent (EC2)
                  |
              HTML Deploy
            (/var/www/html or Nginx)
```

---

## 📦 Prerequisites

* Two AWS EC2 instances:

  * Jenkins Master (Amazon Linux 2 or Ubuntu)
  * Jenkins Agent (same)
* Open ports: `8080` (Jenkins), `22` (SSH)
* GitHub account
* SSH key pair (for GitHub + EC2 agent)
* Security groups configured

---

## 🔧 Step-by-Step Setup

### 🔹 1. Install Jenkins on Master EC2

```bash
# Update and install Java
sudo yum update -y
sudo yum install java-11-openjdk -y

# Add Jenkins repo
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Jenkins
sudo yum install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

Access Jenkins on: `http://<master-public-ip>:8080`

---

### 🔹 2. Install Git and GitHub SSH Key

```bash
sudo yum install git -y
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_jenkins
```

* Add `id_rsa_jenkins.pub` to GitHub → Settings → SSH Keys
* Use `id_rsa_jenkins` as Jenkins Credential (SSH private key)

---

### 🔹 3. Set Up Jenkins Agent (Slave) EC2

On the agent EC2 instance:

```bash
sudo yum install java-11-openjdk -y
```

Add Jenkins master's public key to `/home/ec2-user/.ssh/authorized_keys` for SSH connection.

---

### 🔹 4. Configure Jenkins Agent in UI

1. Jenkins → Manage Jenkins → Nodes → New Node
2. Name: `html-agent`
3. Launch method: **Launch agents via SSH**
4. Host: `<agent-public-ip>`
5. Credentials: Select `jenkins-ssh-key`
6. Remote root dir: `/home/ec2-user/jenkins`
7. Save → Click “Launch agent”

---

## 🖼️ Sample HTML App (index.html)

Create a simple GitHub repo:

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Hai</title>
</head>
<body>
  <h1>Hello Jeetu! 🚀</h1>
  <p>Welcome to your CI/CD pipeline with Jenkins + GitHub + EC2!</p>
</body>
</html>
```

GitHub Repo: [https://github.com/yourusername/html-demo](https://github.com/yourusername/html-demo)

---

## 🔀 Jenkins Pipeline Job (GitHub + Agent Deploy)

1. Jenkins → New Item → Pipeline → Name: `html-deploy-pipeline`
2. In Pipeline script, paste:

```groovy
pipeline {
  agent {
    label 'html-agent'
  }
  stages {
    stage('Clone Repository') {
      steps {
        git credentialsId: 'jenkins-ssh-key', url: 'git@github.com:yourusername/html-demo.git'
      }
    }
    stage('Deploy HTML') {
      steps {
        sh '''
        sudo mkdir -p /var/www/html
        sudo cp index.html /var/www/html/index.html
        '''
      }
    }
  }
}
```

---

## 🔍 Test Deployment

On Agent EC2:

```bash
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

Access HTML page at: `http://<agent-public-ip>`

---

## ✅ CI/CD Flow Summary

1. Push code to GitHub
2. Jenkins master triggers job
3. Jenkins SSHs into agent
4. Agent pulls code and deploys HTML
5. Access via browser — LIVE!

---

## 📍 Security Notes

* Never expose private keys
* Use strong security groups and firewalls
* Use separate credentials for dev/stage/prod

---

## 🎉 You're Done!

You now have:

* 🧠 Jenkins master-agent setup
* ♻️ GitHub-based CI/CD pipeline
* 🌐 HTML deployment to EC2

Happy DevOps-ing! 💻🖠️🚀
