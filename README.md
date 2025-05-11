Here's a well-structured and formatted README for your DevSecOps Pipeline Project: Deploying a Netflix Clone on Kubernetes. This guide encompasses all phases, including infrastructure setup, CI/CD pipeline configuration, security integration, and monitoring.

🚀 Phase 1: Infrastructure Setup
1. Launch EC2 Instance
AMI: Ubuntu 22.04

Instance Type: t2.large

Storage: 25 GB

Security Groups: Allow SSH (22), HTTP (80), HTTPS (443)

Elastic IP: Allocate and associate an Elastic IP to the instance

2. Connect to EC2 Instance

chmod 600 netflix.pem
ssh -i netflix.pem ubuntu@<EC2_PUBLIC_IP>
sudo apt update
3. Clone the Repository

git clone https://github.com/N4si/DevSecOps-Project.git
cd DevSecOps-Project/
4. Install Docker

sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock
docker --version
5. Build and Run the Application
   
docker build -t netflix .
docker images
docker run -d -p 8081:80 <IMAGE_ID>
6. Configure Security Groups
Open the following ports in the EC2 security group:
Medium

8081: Application

8080: Jenkins

9000: SonarQube

7. Validate Application Deployment
Access the application via:

http://<EC2_PUBLIC_IP>:8081
🔐 Phase 2: Security Integration
1. Obtain TMDB API Key
Create an account at The Movie Database

Navigate to Settings > API

Generate a new API key

Note the API key: <API FROM THE SITE>

2. Rebuild Docker Image with API Key

docker build --build-arg TMDB_V3_API_KEY=6ed8c5680fc81eb87d800e62f74b3b05 -t netflix .
docker run -d -p 8081:80 netflix
3. Install SonarQube

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
Access SonarQube at: http://<EC2_PUBLIC_IP>:9000

Default Credentials:

Username: admin

Password: admin

4. Install Trivy

sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
5. Perform Security Scans
Filesystem Scan:

trivy fs .
Docker Image Scan:

docker images
trivy image <IMAGE_ID>
⚙️ Phase 3: CI/CD Pipeline with Jenkins
1. Install Java and Jenkins

sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
java -version

wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
Access Jenkins at: http://<EC2_PUBLIC_IP>:8080

Retrieve the initial admin password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
2. Install Jenkins Plugins
Navigate to Manage Jenkins > Plugins > Available Plugins and install:

Eclipse Temurin Installer

SonarQube Scanner

NodeJS Plugin

OWASP Dependency-Check

Docker, Docker Commons, Docker Pipeline, Docker API, Docker Build Step

3. Configure Jenkins Tools
JDK:

Name: jdk17

Install automatically: Yes

Version: 17.0.8.1+1

NodeJS:

Name: node16

Install automatically: Yes

Version: 16.2.0

SonarQube Scanner:

Name: sonar-scanner

Version: 5.0.1.30006

4. Configure SonarQube in Jenkins
Generate a token in SonarQube:

Navigate to Administration > Security > Users > Tokens

Create a token named Jenkins

In Jenkins:

Navigate to Manage Jenkins > Credentials > System > Global Credentials

Add a new credential:

Kind: Secret text

Secret: <SONARQUBE_TOKEN>

ID: Sonar-token

Description: Sonar-token

Configure SonarQube server:

Navigate to Manage Jenkins > Configure System

Under SonarQube servers, add:

Name: sonar-server

Server URL: http://<EC2_PUBLIC_IP>:9000/

Server authentication token: Sonar-token

5. Create Jenkins Pipeline
Navigate to Jenkins Dashboard > New Item

Enter name: Netflix

Select Pipeline and click OK

In the pipeline configuration, paste the following script:
Medium

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additional
::contentReference[oaicite:208]{index=208}
 
