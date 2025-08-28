pipeline {
    agent any

    environment {
        KUBECONFIG = credentials('4e02ff17-2dd3-4f42-bc24-9ee574aad262')  // <-- ID of kubeconfig credential
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Rubank004/litmus-chaos-gitops.git'
            }
        }

        stage('Apply Chaos Workflow') {
            steps {
                sh """
                kubectl apply -f workflows/kill-card-pod.yml -n litmus
                """
            }
        }

        stage('Verify Workflow') {
            steps {
                sh """
                kubectl get wf -n litmus
                """
            }
        }
    }
}
