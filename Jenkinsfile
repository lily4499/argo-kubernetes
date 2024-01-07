pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('lil_AWS_Access_key_ID')
        AWS_SECRET_ACCESS_KEY = credentials('lil_AWS_Secret_access_key')
        AWS_REGION = "us-east-1"
        EKS_CLUSTER_NAME = 'eks_cluster'
        KUBECONFIG_PATH = '/var/lib/jenkins/.kube/config'
        ARGOCD_APP_NAME = 'lili-app'
    }

    stages {
	stage('Checkout') {
            steps {
           git branch: 'main', url: 'https://github.com/lily4499/argo-kubernetes.git'
  
            }
        }
        stage ("terraform init") {
            steps {
                sh "terraform init" 
            }
        }
  	 stage (" Action") {
            steps {
                sh 'terraform ${action} --auto-approve' 
           }
        }
  	
	stage("Update-kubeconfig") {
            when {
               expression { params.apply }
            }
            steps {
                  sh "aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}"
                 //  sh "kubectl apply -f deployment.yml"
             }
        }
   

        stage('Install ArgoCD in EKS') {
            steps {
                script {
                    // Install ArgoCD in EKS
		    sh "kubectl create namespace argocd"
                    sh "kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml"
                }
            }
        }

        stage('ArgoCD: Deploy Application') {
            steps {
                script {
                    // Wait for ArgoCD pods to be ready

                    sh "kubectl wait --for=condition=Ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s"

                    // Create or sync the application in ArgoCD
                    sh "argocd app create $ARGOCD_APP_NAME --repo https://github.com/lily4499/argo-kubernetes.git --path ./ --dest-server https://kubernetes.default.svc --dest-namespace default"
                    sh "argocd app sync $ARGOCD_APP_NAME"
                }
            }
        }
    }

    post {
        always {
            // Cleanup steps if needed
        }
    }
}
