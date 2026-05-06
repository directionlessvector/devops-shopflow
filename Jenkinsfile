pipeline {
    agent any 

    environment {
        // Now that --ports=8443:8443 is set, this path is open
        MINIKUBE_SERVER = "https://172.17.0.1:8443"
    }

    stages {
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    if [ ! -f ./kubectl ]; then
                        curl -LO "https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                    fi
                    
                    ./kubectl apply -f k8s/deployment.yaml \
                        --server=${MINIKUBE_SERVER} \
                        --insecure-skip-tls-verify \
                        --validate=false
                        
                    ./kubectl apply -f k8s/service.yaml \
                        --server=${MINIKUBE_SERVER} \
                        --insecure-skip-tls-verify \
                        --validate=false
                '''
            }
        }
    }
}