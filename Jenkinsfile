pipeline {
    agent any

    environment {
        // Auto-detect app name from the repo directory name
        APP_NAME       = "${env.JOB_NAME.split('/')[0]}"
        REGISTRY       = 'dockerrepo.icecash.mobi:5000'
        IMAGE          = "${REGISTRY}/sandbox/${APP_NAME}"
        IMAGE_TAG      = "${IMAGE}:${BUILD_NUMBER}"
        IMAGE_LATEST   = "${IMAGE}:latest"
        NAMESPACE      = 'cco-sandbox'
        DOMAIN         = "${APP_NAME}.sandbox.digittal.mobi"

        // Cluster credentials — adjust if targeting a different cluster
        KUBE_CREDENTIALS_ID = 'hetzner-credentials'
        KUBE_SERVER_URL     = 'https://136.243.89.211:16443'
    }

    triggers {
        // GitHub webhook trigger — fires on push
        githubPush()
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building ${IMAGE_TAG} ..."
                    def app = docker.build("${IMAGE_TAG}", ".")
                    app.push()
                    app.push('latest')
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: KUBE_CREDENTIALS_ID, serverUrl: KUBE_SERVER_URL]) {
                        // Apply k8s manifests from the repo (deploy/k8s/ directory)
                        // Substitute environment variables in the templates
                        sh """
                            export APP_NAME="${APP_NAME}"
                            export IMAGE_TAG="${IMAGE_TAG}"
                            export NAMESPACE="${NAMESPACE}"
                            export DOMAIN="${DOMAIN}"

                            # Process and apply each manifest template
                            for f in deploy/k8s/*.yaml; do
                                envsubst < "\$f" | kubectl apply -n ${NAMESPACE} -f -
                            done
                        """

                        // Verify rollout
                        sh """
                            kubectl rollout status deployment/${APP_NAME} \
                                -n ${NAMESPACE} --timeout=120s
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Deployed successfully: https://${DOMAIN}"
        }
        failure {
            echo "Build or deploy failed for ${APP_NAME}"
        }
    }
}
