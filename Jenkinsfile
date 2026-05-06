pipeline {
    agent any

    stages {

        stage('Clone Check') {
            steps {
                sh 'ls'
            }
        }

        stage('Docker Check') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t shopflow-test .'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml -n default'
                sh 'kubectl apply -f k8s/service.yaml -n default'
            }
        }
    }
}