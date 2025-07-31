pipeline {
    agent { label 'maven_slave' }  // Use the slave node with this label

    environment {
        DOCKER_IMAGE = "nikithamanvi/financeapp:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/nikimanvi/star-agile-banking-finance.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker Image: ${DOCKER_IMAGE}"
                sh """
                    docker build -t ${DOCKER_IMAGE} .
                    docker tag ${DOCKER_IMAGE} nikithamanvi/financeapp:latest
                    docker image ls
                """
            }
        }
        stage('Login to Dockerhub') {
            steps {
                echo 'Logging into DockerHub...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerloginid', 
                    usernameVariable: 'DOCKERHUB_CREDENTIALS_USR', 
                    passwordVariable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                        
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                }
            }
        }

        stage('Publish the Image to Dockerhub') {
            steps {
                echo "Publishing Docker Image: ${DOCKER_IMAGE}"
                sh """
                    docker push ${DOCKER_IMAGE}
                    docker push nikithamanvi/financeapp:latest
                """
            }
        }
    }
}
