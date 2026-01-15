# jenkins-docker-demo

PART 1: AWS EC2 SETUP (FROM ZERO)
STEP 1: Create Ubuntu EC2 Instance

Login to AWS Console

Go to EC2 → Launch Instance

Choose:

AMI: Ubuntu 22.04

Instance type: t2.micro (FREE TIER)

Create Key Pair

Name: jenkins-key

Download .pem file

Network settings:

Allow:

SSH (22)

HTTP (80)

Custom TCP (8080)

Click Launch Instance

STEP 2: Connect to EC2
chmod 400 jenkins-key.pem
ssh -i jenkins-key.pem ubuntu@<EC2_PUBLIC_IP>


Now you are inside your AWS Ubuntu server 🎉

🟢 PART 2: INSTALL DOCKER (REQUIRED)
STEP 3: Install Docker
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker


Give permission:

sudo usermod -aG docker ubuntu
newgrp docker


Verify:

docker --version

🟢 PART 3: RUN JENKINS USING DOCKER
STEP 4: Run Jenkins Container
docker run -d \
-p 8080:8080 \
-p 50000:50000 \
-v jenkins_home:/var/jenkins_home \
--name jenkins \
jenkins/jenkins:lts

STEP 5: Get Jenkins Password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

STEP 6: Open Jenkins UI

Open browser:

http://<EC2_PUBLIC_IP>:8080


Steps:

Paste password

Install Suggested Plugins

Create admin user

Finish setup

🟢 PART 4: PREPARE SAMPLE APPLICATION
STEP 7: Install Git & Node.js
sudo apt install git -y
sudo apt install nodejs npm -y

STEP 8: Create App Folder
mkdir jenkins-demo
cd jenkins-demo

STEP 9: Create Node App
app.js
nano app.js

console.log("Hello from Jenkins CI/CD Pipeline 🚀");

package.json
nano package.json

{
  "name": "jenkins-demo",
  "version": "1.0.0",
  "scripts": {
    "test": "echo Test Passed",
    "start": "node app.js"
  }
}

STEP 10: Create Dockerfile
nano Dockerfile

FROM node:18
WORKDIR /app
COPY . .
CMD ["npm", "start"]

🟢 PART 5: PUSH CODE TO GITHUB
STEP 11: Create GitHub Repository

Repo name: jenkins-docker-demo

Public

STEP 12: Push Code
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<YOUR_USERNAME>/jenkins-docker-demo.git
git push -u origin main

🟢 PART 6: JENKINS CONFIGURATION
STEP 13: Allow Jenkins to Use Docker
docker exec -u root jenkins usermod -aG docker jenkins
docker restart jenkins

STEP 14: Create Jenkins Pipeline

Jenkins Dashboard → New Item

Name: jenkins-docker-pipeline

Select Pipeline

Click OK

STEP 15: Jenkins Pipeline Script

Paste this 👇
(Replace GitHub URL)

pipeline {
    agent any

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/YOUR_USERNAME/jenkins-docker-demo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-demo-app .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                docker stop demo-container || true
                docker rm demo-container || true
                docker run --name demo-container jenkins-demo-app
                '''
            }
        }
    }

    post {
        success {
            echo "BUILD SUCCESS 🎉"
        }
        failure {
            echo "BUILD FAILED ❌"
        }
    }
}

🟢 PART 7: RUN PIPELINE
STEP 16: Build Pipeline

Click Build Now

Open Console Output

Expected output:

Test Passed
Hello from Jenkins CI/CD Pipeline 🚀


🎉 CI/CD PIPELINE WORKING!
