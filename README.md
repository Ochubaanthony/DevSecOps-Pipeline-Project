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
clear
<IMAGE_ID>
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
2. Create account with https://www.themoviedb.org/
Create an account at The Movie Database

Navigate to Settings > API

Generate a new API key

Note the API key: <API FROM THE SITE>

Run this to find which container is using that port
docker ps
docker stop <container_id>
docker rm <container_id>
validate ip<8081> to view complete netflix 


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

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

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
dropdown Install from adoption.net
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

Navigate to Administration > Security > Users > Tokens>Generate
Copy the token and go back to Jenkins page 

Create a token named Jenkins

In Jenkins:

Navigate to Manage Jenkins > Credentials > System > Global Credentials

Add a new credential:

Kind: Secret text

Secret: <SONARQUBE_TOKEN>

ID: Sonar-token

Description: Sonar-token

Configure SonarQube server:
create


Navigate to Manage Jenkins > Configure System

Under SonarQube servers, add:

Name: sonar-server

Server URL: http://<EC2_PUBLIC_IP>:9000/

Server authentication token: Sonar-token


Steps 
Go back to Manager Jenkins
Click on Tools
Click on SonarQube Scanner Installation 
Click on Add SonarQube scann 
Name it : sonar-scanner
Version : SonarQube Scanner 5.0.1.30006
Click on Apply and save 


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




Steps
Click on Manage Jenkins
Click on Credentials
Click on System 
Click on Global credentials 
Click on Add Credentials 
Kind: 
	username for my dockerhub is tonymore
	password is Oluchi1995
ID : docker
Description : docker
Click on Create



 Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.


  Create Jenkins Pipeline
Navigate to Jenkins Dashboard > New Item

Enter name: Netflix

Select Pipeline and click OK

In the pipeline configuration, paste the following script:
Medium

 
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
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
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourapikey> -t netflix ."
                       sh "docker tag netflix nasi101/netflix:latest "
                       sh "docker push nasi101/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nasi101/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
    }
}


If you get docker login failed errorr

sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins


GO TO TO SonarQube web Tab
Click on Project
Click Manually
Give the name of project display as : Netflix
Click on SetUp

Validate
Click on Locally
Click on Generate
Click on Continue
Click on Other
Click on Linux
The SonarQube web tab will show you that all PASSED





Phase 4: Monitoring
Create T2.medium instance for Monitoring Server
Security group “ All SSH traffic from and Allow HTTPS traffic from the internet”
Storage should be 20GB
Lauch instance 

Install Prometheus and Grafana:
Set up Prometheus and Grafana to monitor your application.
Installing Prometheus

First, create a dedicated Linux user for Prometheus and download Prometheus:”AWS SERVER”

sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz

RUN     ls   “ to see Prometheus is install”


Extract Prometheus files, move them, and create directories:

tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml


RUN
ls
cd /etc/prometheus/
ls

cd /usr/local/bin/
ls

Verify if they is Service
sudo service prometheus             “prometheus: unrecognized service”
 ls -la /etc/prometheus/

 Set ownership for directories:
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
 ls -la /etc/prometheus/

Create a systemd unit configuration file for Prometheus:
sudo nano /etc/systemd/system/prometheus.service

Add the following content to the prometheus.service file:
PASTE CODE BELOW

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target



Enable and start Prometheus:
sudo systemctl enable prometheus
sudo systemctl start prometheus

Verify Prometheus's status:
sudo systemctl status prometheus

Add Port 9090 on the Server security  Group
98.85.188.61:9090

On the Prometheus webpage
Click on status
Click on target


Step 
Installing Node Exporter:

Create a system user for Node Exporter and download Node Exporter:
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

cd ~
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

ls
You see : node_exporter-1.6.1.linux-amd64.tar.gz  prometheus-2.47.1.linux-amd64  prometheus-2.47.1.linux-amd64.tar.gz

Extract Node Exporter files, move the binary, and clean up:

tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*

Create a systemd unit configuration file for Node Exporter:
sudo nano /etc/systemd/system/node_exporter.service

PASTE

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target


sudo useradd --no-create-home --shell /bin/false node_exporter

sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

Enable and start Node Exporter:
sudo systemctl daemon-reload
sudo systemctl restart node_exporter

Verify the Node Exporter's status:
sudo systemctl status node_exporter



cd /etc/prometheus/
ls
cat prometheus.yml
sudo nano prometheus.yml

Add this below
 - job_name: "node_exporter"
    static_configs:
      - targets: ["54.91.151.246:9100"]               "this ip address for monitoring server"

Check the validity of the configuration file: 
promtool check config /etc/prometheus/prometheus.yml

Reload the Prometheus configuration without restarting:
curl -X POST http://localhost:9090/-/reload                         "open port 9100 on the monitoring server, reload prometheus web tap"

