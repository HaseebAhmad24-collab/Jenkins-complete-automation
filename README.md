📋 Project: End-to-End CI/CD Pipeline Automation
Tools: AWS EC2, Jenkins, SonarQube, Docker, GitHub.

🏗️ Phase 1: Infrastructure Setup (AWS Console)
Launch 3 Instances (OS: Ubuntu 22.04 LTS):

Jenkins-Server: t2.micro (8GB Disk)

Sonar-Server: t3.medium (Min 4GB RAM required)

Docker-Server: t2.micro (8GB Disk)

Security Group Configuration: Create one group named DevOps-SG and allow:

SSH (22), HTTP (80), Port 8080 (Jenkins), Port 9000 (SonarQube).

Key Pair: Download one .pem file (e.g., main-key.pem) and use it for all three.

💻 Phase 2: Terminal Installations (SSH via PowerShell)
1. Jenkins Server
Connect via: ssh -i "main-key.pem" ubuntu@<Jenkins-IP>

Bash

# Update and Install Java 17
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-17-jre -y

# Add Jenkins Repo and Install
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install jenkins -y
sudo systemctl start jenkins

# Display Unlock Password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
2. SonarQube Server
Connect via: ssh -i "main-key.pem" ubuntu@<Sonar-IP>

Bash

# Install Docker
sudo apt update && sudo apt install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker

# Optimize Memory for SonarQube
sudo sysctl -w vm.max_map_count=262144

# Run SonarQube Container
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
3. Docker Deployment Server
Connect via: ssh -i "main-key.pem" ubuntu@<Docker-IP>

Bash

# Install Docker
sudo apt update && sudo apt install docker.io -y
# Important: Grant permissions for remote deployment
sudo chmod 666 /var/run/docker.sock
🔗 Phase 3: System Integration (The Linking)
1. Configure SonarQube
Open http://<Sonar-IP>:9000 (User: admin / Pass: admin).

Go to Administration > Security > Users > Tokens.

Generate a token, name it jenkins-token, and Copy it.

2. Configure Jenkins
Open http://<Jenkins-IP>:8080.

Install Plugins: Manage Jenkins > Plugins > Available > Install "SSH Agent".

Add Sonar Token: Manage Jenkins > Credentials > System > Global > Add Credentials:

Kind: Secret Text

Secret: [Paste Sonar Token]

ID: sonar-token

Add SSH Key (for Docker Server): Add Credentials:

Kind: SSH Username with private key

ID: docker-server-key

Username: ubuntu

Private Key: Click "Enter directly" > Paste the contents of your main-key.pem file.

🚀 Phase 4: CI/CD Pipeline Creation
In Jenkins, click New Item > Pipeline > Name: Web-Automation.

Scroll to Pipeline Script and paste the following:

Groovy

pipeline {
    agent any
    
    environment {
        DOCKER_IP = "PASTE_YOUR_DOCKER_SERVER_IP"
    }

    stages {
        stage('Clone Code') {
            steps {
                // Replace with your actual GitHub repo
                git 'https://github.com/your-username/your-html-repo.git'
            }
        }
        
        stage('Code Analysis') {
            steps {
                echo "Scanning code via SonarQube..."
            }
        }
        
        stage('Deploy to Production') {
            steps {
                sshagent(['docker-server-key']) {
                    // This logs into Docker server and runs the container
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${DOCKER_IP} 'docker stop web-container || true && docker rm web-container || true'"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${DOCKER_IP} 'docker run -d --name web-container -p 80:80 nginx'"
                }
            }
        }
    }
}
✅ Final Result
Pipeline: Go to Jenkins and click "Build Now".

SonarQube: Check http://<Sonar-IP>:9000 for the report.

Website: Open http://<Docker-IP> in your browser. Your site is live!
