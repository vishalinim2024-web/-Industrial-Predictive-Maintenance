pipeline {
    agent any

    environment {
        REGISTRY_CREDENTIALS = 'docker-hub-credentials'
        IMAGE_NAME           = 'industrial/predictive-maintenance-dashboard'
        KUBE_CONFIG_CRED     = 'k8s-cluster-config'
        APP_RELEASE_NAME     = 'predictive-maintenance-system'
        NAMESPACE            = 'production-monitoring'
    }

    stages {
        stage('Lint & Source Code Validation') {
            steps {
                echo 'Analyzing visual assets and HTML source profiles...'
                sh 'grep -q "Live Asset Telemetry" index.html'
                sh 'grep -q "Priority Action Board" index.html || true' 
            }
        }

        stage('Secure Multi-Stage Container Build') {
            steps {
                echo 'Triggering Docker construction algorithms...'
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                sh "docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest"
            }
        }

        stage('Container Asset Deployment') {
            steps {
                echo 'Updating cluster orchestration layers with zero-downtime rolling update...'
                withKubeConfig([credentialsId: "${KUBE_CONFIG_CRED}"]) {
                    // Apply static objects configurations setup definitions
                    sh "kubectl apply -f deployment.yaml -n ${NAMESPACE}"
                    // Rollout new build image reference update to components
                    sh "kubectl set image deployment/${APP_RELEASE_NAME} web-dashboard=${IMAGE_NAME}:${BUILD_NUMBER} -n ${NAMESPACE}"
                }
            }
        }

        stage('Active Post-Deployment Health Check') {
            options {
                timeout(time: 5, unit: 'MINUTES')
            }
            steps {
                echo 'Polling service container logs and endpoints...'
                withKubeConfig([credentialsId: "${KUBE_CONFIG_CRED}"]) {
                    script {
                        try {
                            // Watch rollouts evaluate parameters smoothly
                            sh "kubectl rollout status deployment/${APP_RELEASE_NAME} -n ${NAMESPACE}"
                        } catch (Exception err) {
                            echo "Anomaly or build regression detected! Initializing pipeline safety rollbacks..."
                            sh "kubectl rollout undo deployment/${APP_RELEASE_NAME} -n ${NAMESPACE}"
                            error("Deployment degraded. System rolled back to previous stable container target.")
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Clearing lingering disk assets from local execution worker nodes...'
            cleanWs()
        }
    }
}
