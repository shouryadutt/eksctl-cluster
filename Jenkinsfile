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
                script {
                    def clusterExists = sh(script: 'aws eks describe-cluster --cluster-name realtime-project --region us-west-2', returnStatus: true) != 0
                    if (clusterExists){
                        echo "Cluster Already Exists"  
                    }
                    else{
                        echo "Creating Cluster"
                        sh "eksctl create cluster --config-file=cluster.yaml"
                    }
                }
            }
        }
        stage('Update AWS AUTH Config') {
            steps {
                sh 'kubectl patch configmap aws-auth -n kube-system --patch "$(cat aws-auth-cm.yaml)"'
            }
        }
    }
}