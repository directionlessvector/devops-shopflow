pipeline {
    agent any

    stages {

        stage('Clone Check') {
            steps {
                sh 'ls'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {

                    def scannerHome = tool 'sonar-scanner'

                    withCredentials([
                        string(
                            credentialsId: 'sonarqube-token',
                            variable: 'SONAR_TOKEN'
                        )
                    ]) {

                        withSonarQubeEnv('sonarqube') {

                            sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
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