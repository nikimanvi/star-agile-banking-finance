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
    }
}
