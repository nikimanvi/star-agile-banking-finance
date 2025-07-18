pipeline {
    agent any

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

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerloginid', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes via SSH Publisher') {
    steps {
        sshPublisher(publishers: [
            sshPublisherDesc(
                configName: 'Kubernetes_Master',
                transfers: [
                    sshTransfer(
                        sourceFiles: 'k8s/*.yaml',
                        removePrefix: '',
                        remoteDirectory: '.',
                        execCommand: 'kubectl apply -f k8s/deployment.yaml && kubectl apply -f k8s/service.yaml',
                        execTimeout: 120000
                    )
                ],
                verbose: true
            )
        ])
    }
}

    post {
        success {
            echo '✅ Deployment completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
