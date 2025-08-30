pipeline {
    agent any

    environment {
        KUBECTL = "./kubectl"   // Use local binary
        NAMESPACE = "litmus"    // Namespace where litmus is installed
        EXPERIMENT = "kill-card-pod.yaml" // Your chaos experiment yaml file
    }

    stages {

        stage('Checkout Code') {
            steps {
                // Replace with your GitHub repo
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
            mv kubectl /usr/local/bin/
            kubectl version --client
        '''
    }
}

        stage('Apply Chaos Experiment') {
            steps {
                sh '''
                echo "Applying chaos experiment..."
                ${KUBECTL} apply -f ${EXPERIMENT} -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify Chaos Result') {
            steps {
                script {
                    def result = sh(
                        script: "${KUBECTL} get chaosresult -n ${NAMESPACE} -o jsonpath='{.items[0].status.experimentStatus.verdict}'",
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
