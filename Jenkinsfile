pipeline {
    agent any

    environment {
        PATH = "$HOME/bin:$PATH"          // Ensure kubectl is in PATH
        KUBECTL = "kubectl"               // Use directly
        NAMESPACE = "litmus"
        EXPERIMENT = "kill-card-pod.yaml"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Rubanrk004/litmus-chaos-gitops.git'
            }
        }

        stage('Install kubectl') {
            steps {
                sh '''
                    echo "Downloading kubectl..."
                    KUBE_VERSION=$(curl -Ls https://dl.k8s.io/release/stable.txt)
                    echo "Latest kubectl version: $KUBE_VERSION"
                    curl -LO "https://dl.k8s.io/${KUBE_VERSION}/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mkdir -p $HOME/bin
                    mv kubectl $HOME/bin/
                    kubectl version --client
                '''
            }
        }

        stage('Apply Chaos Experiment') {
            steps {
                sh '''
                    echo "Applying chaos experiment..."
                    kubectl apply -f ${EXPERIMENT} -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Chaos Result') {
            steps {
                script {
                    def result = sh(
                        script: "kubectl get chaosresult -n ${NAMESPACE} -o jsonpath='{.items[0].status.experimentStatus.verdict}'",
                        returnStdout: true
                    ).trim()

                    echo "Chaos Experiment Verdict: ${result}"

                    if (result != "Pass") {
                        error("Chaos Experiment Failed with Verdict: ${result}")
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        success {
            echo "✅ Chaos Experiment PASSED"
        }
        failure {
            echo "❌ Chaos Experiment FAILED"
        }
    }
}
