pipeline {
    agent any

    stages {
        stage('Checking the Config') {
            steps {
                sh "eksctl create cluster --config-file=cluster.yaml --dry-run"
            }
        }
        stage('Creating Cluster') {
            steps {
                sh "eksctl create cluster --config-file=cluster.yaml"
            }
        }
    }
}