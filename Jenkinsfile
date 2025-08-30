pipeline {
    agent any

    environment {
        KUBECONFIG = credentials('4e02ff17-2dd3-4f42-bc24-9ee574aad262')
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
            steps {
                sh 'kubectl apply -f workflows/kill-card-pod.yml -n litmus'
            }
        }

        stage('Verify Workflow') {
            steps {
                sh 'kubectl get chaosengine -n litmus'
                sh 'kubectl get chaosresult -n litmus'
            }
        }
    }
}
