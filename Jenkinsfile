pipeline {
    agent any

    stages {
        stage('Creating Cluster') {
            steps {
                echo "Creating Cluster"
                sh "eksctl create cluster --config-file=cluster.yaml"
            }
        }
    }
}
