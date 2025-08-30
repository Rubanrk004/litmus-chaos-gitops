pipeline {
    agent any

    environment {
        NAMESPACE = "litmus"                        // Namespace where Litmus is installed
        EXPERIMENT = "workflows/kill-card-pod.yml"  // Path to chaos workflow in repo
        KUBECTL = "$HOME/bin/kubectl"               // Absolute kubectl path
        KUBECONFIG = "/root/.kube/config"           // Adjust if different in Jenkins agent
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

                    echo "Verifying kubectl..."
                    $HOME/bin/kubectl version --client
                '''
            }
        }

        stage('Apply Chaos Experiment') {
            steps {
                sh '''
                    echo "Applying chaos experiment..."
                    $HOME/bin/kubectl apply -f ${EXPERIMENT} -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Chaos Result') {
            steps {
                script {
                    def result = sh(
                        script: "$HOME/bin/kubectl get chaosresult -n ${NAMESPACE} -o jsonpath='{.items[0].status.experimentStatus.verdict}'",
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
