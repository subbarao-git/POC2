pipeline {
    agent any

    environment {
        IMAGE_NAME = "subbu2712/java-demo"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/subbarao-git/POC2.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([string(credentialsId: 'Kalpana@2810', variable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u subbu2712 --password-stdin'
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker stop java-app || true'
                sh 'docker rm java-app || true'
                sh 'docker run -d -p 8080:8080 --name java-app $IMAGE_NAME'
            }
        }
    }
}
