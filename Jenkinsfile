pipeline {
    agent any

    environment {
        // Namespace where Litmus is installed
        LITMUS_NAMESPACE = "litmus"

        // Namespace where your application is deployed
        APP_NAMESPACE = "microservices"

        // Path to the chaos experiment YAML inside repo
        EXPERIMENT = "workflows/kill-card-pod.yml"

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
                    echo "📂 Checking repo file structure..."
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

                    echo "✅ Verifying kubectl..."
                    $HOME/bin/kubectl version --client
                '''
            }
        }

stage('Apply Chaos Experiment') {
    steps {
        withCredentials([file(credentialsId: '4e02ff17-2dd3-4f42-bc24-9ee574aad262', variable: 'KUBECONFIG')]) {
            sh '''
                echo "⚡ Applying chaos experiment..."
                export KUBECONFIG=$KUBECONFIG   # 👈 tell kubectl to use this file
                $HOME/bin/kubectl apply -f ${EXPERIMENT} -n ${LITMUS_NAMESPACE}
            '''
        }
    }
}

stage('Verify Chaos Result') {
    steps {
        withCredentials([file(credentialsId: '4e02ff17-2dd3-4f42-bc24-9ee574aad262', variable: 'KUBECONFIG')]) {
            sh '''
                echo "⏳ Waiting for chaos result..."
                sleep 30

                export KUBECONFIG=$KUBECONFIG   # 👈 again here
                verdict=$($HOME/bin/kubectl get chaosresult -n ${APP_NAMESPACE} -o jsonpath='{.items[0].status.experimentStatus.verdict}')
                
                echo "📝 Chaos Experiment Verdict: $verdict"

                if [ "$verdict" != "Pass" ]; then
                  echo "❌ Chaos Experiment Failed with Verdict: $verdict"
                  exit 1
                fi
            '''
        }
    }
}

    post {
        always {
            echo "📌 Pipeline completed."
        }
        success {
            echo "✅ Chaos Experiment PASSED"
        }
        failure {
            echo "❌ Chaos Experiment FAILED"
        }
    }
}
