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
    }
}