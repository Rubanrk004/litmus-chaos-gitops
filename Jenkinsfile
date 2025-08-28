pipeline {
    agent any

    environment {
        KUBECONFIG = credentials('4e02ff17-2dd3-4f42-bc24-9ee574aad262')  // Kubeconfig stored in Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Rubanrk004/litmus-chaos-gitops.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Apply Chaos Workflow') {
            agent {
                kubernetes {
                    yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
"""
                }
            }
            steps {
                container('kubectl') {
                    sh 'kubectl apply -f workflows/kill-card-pod.yml'
                }
            }
        }

        stage('Verify Workflow') {
            agent {
                kubernetes {
                    yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
"""
                }
            }
            steps {
                container('kubectl') {
                    sh 'kubectl get chaosengine -n litmus'
                }
            }
        }
    }
}
