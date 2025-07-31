pipeline {
    agent { label 'maven_slave' }  // Slave node with Maven and Docker installed

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

        stage('Login to DockerHub') {
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

        stage('Publish the Image to DockerHub') {
            steps {
                echo "Publishing Docker Image: ${DOCKER_IMAGE}"
                sh """
                    docker push ${DOCKER_IMAGE}
                    docker push nikithamanvi/financeapp:latest
                """
            }
        }

stage('Deploy to EKS') {
    steps {
        script {
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: 'eks-master',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'deployment.yaml',
                                remoteDirectory: '/home/ec2-user/deploy',
                                remoteDirectorySDF: true,     // ✅ This makes it use ABSOLUTE path
                                removePrefix: '',
                                flatten: true,
                                execCommand: 'ls -l /home/ec2-user/deploy && kubectl apply -f /home/ec2-user/deploy/deployment.yaml',
                                execTimeout: 120000
                            )
                        ],
                        usePromotionTimestamp: false,
                        verbose: true
                    )
                ]
            )
        }
    }
}

    }
}
