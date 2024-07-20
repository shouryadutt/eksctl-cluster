
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
                    } else {
                        echo "Creating Cluster"
                        sh "eksctl create cluster --config-file=cluster.yaml"
                    }
                }
            }
        }
        stage('Update Kube-config') {
            steps {
                sh "aws eks update-kubeconfig --name realtime-project --region us-west-2"
            }
        }
        stage('Apply ClusterRole and ClusterRoleBinding') {
            steps {
                sh 'kubectl apply -f cluster-role.yml'
            }
        }
        stage('Update AWS AUTH Config') {
            steps {
                sh 'kubectl patch configmap aws-auth -n kube-system --patch "$(cat aws-auth-cm.yaml)"'
            }
        }
        stage('Verify ArgoCD Namespace') {
            steps {
                script {
                    def nsExists = sh(script: "kubectl get namespace argocd --ignore-not-found", returnStatus: true) != 0
                    if (!nsExists) {
                        sh 'kubectl create namespace argocd'
                    } else {
                        echo 'Namespace argocd already exists'
                    }
                }
            }
        }

        stage('Install ArgoCD') {
            steps {
                script {
                    def argocdInstalled = sh(script: "kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server --ignore-not-found", returnStatus: true) != 0
                    if (!argocdInstalled) {
                        sh '''
                            kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
                            kubectl wait --for=condition=available --timeout=600s deployment -l app.kubernetes.io/name=argocd-server -n argocd
                        '''
                    } else {
                        echo 'ArgoCD is already installed'
                    }
                }
            }
        }
        stage('Expose ArgoCD with LoadBalancer') {
            steps {
                script {
                    sh '''
                        kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
                        
                        while [ -z "$(kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')" ]; do
                          echo "Waiting for LoadBalancer IP..."
                          sleep 30
                        done
                        
                         kubectl get svc argocd-server -n argocd

                        export ARGOCD_LB_IP=$(kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                        echo "ArgoCD LoadBalancer IP: $ARGOCD_LB_IP"
                    '''
                }
            }
        }
        stage('Configure ArgoCD') {
            steps {
                script {
                    sh '''
                        export ARGOCD_LB_IP=$(kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
                        
                        # Log in to ArgoCD
                        argocd login $ARGOCD_LB_IP --username admin --password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo) --insecure

                        # Change the default admin password
                        argocd account update-password --account admin --current-password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo) --new-password mynewpassword
                    '''
                }
            }
        }
    }
}
