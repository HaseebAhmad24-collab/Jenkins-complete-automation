# 📋 Project: Full Stack CI/CD Automation Guide

## 🏗️ Phase 1: Infrastructure Setup (AWS Console)
1. **Launch 3 Instances** (OS: Ubuntu 22.04 LTS):
   - **Jenkins-Server:** `t2.micro`
   - **Sonar-Server:** `t3.medium`
   - **Docker-Server:** `t2.micro`
2. **Security Group:** Open Ports `22, 80, 8080, 9000`.

---

## 💻 Phase 2: Terminal Installations

### 1. Jenkins Server Setup
```bash
# Update and Install Java
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-17-jre -y

# Install Jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc [https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key](https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key)
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] [https://pkg.jenkins.io/debian-stable](https://pkg.jenkins.io/debian-stable) binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install jenkins -y
sudo systemctl start jenkins

# Get Initial Password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
2. SonarQube Server Setup
# Install Docker
sudo apt update && sudo apt install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker

# System Optimization
sudo sysctl -w vm.max_map_count=262144

# Start SonarQube
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
3. Docker Deployment Server Setup
# Install Docker
sudo apt update && sudo apt install docker.io -y

# Fix Permissions for Jenkins Access
sudo chmod 666 /var/run/docker.sock
🔗 Phase 3: Integration & Credentials
A. SonarQube Token
Login admin/admin.

Administration > Security > Users > Tokens.

Generate & Copy Token.

B. Jenkins Credentials Management
Sonar Token: Kind: Secret Text | ID: sonar-token

Deployment Key: Kind: SSH Username with private key | ID: docker-server-key | Username: ubuntu | Private Key: [Paste .pem content]

🚀 Phase 4: Jenkins Pipeline Script
pipeline {
    agent any
    environment {
        DOCKER_IP = "YOUR_DOCKER_IP"
    }
    stages {
        stage('Clone') {
            steps { git '[https://github.com/your-repo.git](https://github.com/your-repo.git)' }
        }
        stage('Scan') {
            steps { echo "Analyzing code..." }
        }
        stage('Deploy') {
            steps {
                sshagent(['docker-server-key']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${DOCKER_IP} 'docker run -d -p 80:80 nginx'"
                }
            }
        }
    }
}
---

### **Important Tip:**
Jab aap Notepad ya kisi code editor mein ye file save karein, to extension hamesha **`.md`** rakhein. Agar aap ise GitHub par upload karenge ya kisi Markdown viewer mein dekhenge, to ye bilkul professional aur alag alag blocks mein dikhayi degi.

**Kya aap chahte hain ke main Pipeline script ko mazeed behtar kar doon taaki wo actually aapka HTML code deploy kare?**
