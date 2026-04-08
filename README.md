# POC2
**Documentation to create a ci/cd pipeline using github, jenkins, docker and aws ec2 instance**.

**1. Prerequisites**

**1.1 GitHub**

GitHub account
Application source code repository
Dockerfile in repository

**1.2 AWS EC2**

EC2 instance (Amazon Linux / Ubuntu)

Open ports:

    22 – SSH
    
    8080 – Jenkins
    
    80/443 – Application
    
Key Pair (PEM file)

**1.3 Jenkins Server**

Jenkins installed on EC2 or separate server

Jenkins user with sudo access

**1.4 Tools Installed**

**Tool                Purpose**

Git                Source control

Jenkins            CI/CD Orchestrator

Docker             Containerization

AWS CLI            AWS operations

**2. Jenkins Installation on EC2**

**2.1 Install Java**

Shellsudo yum install java-17-amazon-corretto -yShow more lines

**2.2 Install Jenkins**

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key

sudo yum install jenkins -y

**2.3 Start Jenkins**

sudo systemctl start jenkins

sudo systemctl enable jenkins

**Access Jenkins:**

http://<EC2-Public-IP>:8080


**3. Docker Installation on EC2**

sudo yum install docker -y

sudo systemctl start docker

sudo systemctl enable docker

sudo usermod -aG docker jenkins

sudo usermod -aG docker ec2-user

**Restart Jenkins:**

sudo systemctl restart jenkins

**4. GitHub Configuration**

**4.1 Repository Structure (Example)**
.

├── Dockerfile

├── Jenkinsfile

├── app/

├── requirements.txt

└── README.md

**4.2 Sample Dockerfile**

FROM eclipse-temurin:17-jdk-jammy

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","app.jar"]

**5. Jenkins Configuration**

**5.1 Required Jenkins Plugins**

**Install the following:**

Git

GitHub Integration

Credentials Binding

**6. Jenkins Credentials Setup**

**6.1 GitHub Credentials**

Type: Username & Token

ID: github-creds

**6.2 Docker Registry Credentials**

Type: Username & Password

ID: dockerhub-creds

**7. Jenkins Pipeline (Jenkinsfile)**


pipeline {

    agent any

    environment {
        IMAGE_NAME = "yourdockerhubusername/app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/username/repo.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                    sh """
                        docker pull $IMAGE_NAME:latest
                        docker stop app || true
                        docker rm app || true
                        docker run -d --name app -p 8081:8080 $IMAGE_NAME:latest
                    '
                    """
            }
        }
    }
}

**8. Deployment Verification**

**8.1 Check Jenkins Console**

All stages should be SUCCESS

**8.2 Verify Application**

http://<EC2-PUBLIC-IP>

**9. Security Best Practices**

Use IAM Roles instead of AWS keys

Restrict EC2 security group access

Use private Docker registry

Enable Jenkins authentication

Rotate credentials regularly


**10. Common Issues & Fixes**
**Issue                                  Cause                              Fix**

Docker permission denied      Jenkins not in docker group        Restart after usermod

Webhook not triggering                Firewall                      Open port 8080

SSH timeout                          Wrong key                        Verify PEM

**11. CI/CD Enhancement Ideas**

Blue/Green deployment

Auto-scaling EC2

Use ECR instead of DockerHub

Add SonarQube for code quality

Add Slack/Email notifications

**12. Summary**

**This CI/CD pipeline:**

✅ Automates builds

✅ Containers application

✅ Deploys reliably to AWS EC2

✅ Reduces manual errors
