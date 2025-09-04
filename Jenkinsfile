pipeline {
    agent any

    environment {
        // Namespace where Litmus is installed
        LITMUS_NAMESPACE = "litmus"

        // Namespace where your application is deployed
        APP_NAMESPACE = "microservices"

        // Path to the chaos experiment YAML inside repo
        EXPERIMENT = "litmus/9cd336f5-a916-4d9b-a352-5a865e6f6b88/network-latency-cart.yaml"

        KUBECTL = "$HOME/bin/kubectl"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Rubanrk004/litmus-chaos-gitops.git'
            }
        }

        stage('Debug Repo Structure') {
            steps {
                sh '''
                    echo "Checking repo file structure..."
                    pwd
                    ls -R
                '''
            }
        }

        stage('Install kubectl') {
            steps {
                sh '''
                    echo "⬇️ Downloading kubectl..."
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
                withCredentials([file(credentialsId: '4e02ff17-2dd3-4f42-bc24-9ee574aad262', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        echo "Setting kubeconfig..."
                        export KUBECONFIG=$KUBECONFIG_FILE
                        echo "Using kubeconfig: $KUBECONFIG"

                        echo "Checking cluster access..."
                        $HOME/bin/kubectl get ns

                        echo "Applying chaos experiment..."
                        $HOME/bin/kubectl apply -f ${EXPERIMENT} -n ${LITMUS_NAMESPACE}
                    '''
                }
            }
        }

        stage('Verify Chaos Experiment') {
            steps {
                withCredentials([file(credentialsId: '4e02ff17-2dd3-4f42-bc24-9ee574aad262', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        echo "Verifying chaos workflow run..."
                        export KUBECONFIG=$KUBECONFIG_FILE
                        $HOME/bin/kubectl get wf -n ${LITMUS_NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        success {
            echo "Chaos Experiment PASSED"
        }
        failure {
            echo "Chaos Experiment FAILED"
        }
    }
}
