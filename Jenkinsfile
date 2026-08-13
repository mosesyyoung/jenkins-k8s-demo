pipeline {

    agent any

    environment {
        HARBOR_REGISTRY = '192.168.56.101:8088'
        HARBOR_PROJECT  = 'k8s-lab'
        IMAGE_NAME      = 'jenkins-k8s-demo'

        K8S_NAMESPACE = 'production'
        KUBECONFIG    = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Prepare') {
            steps {
                script {
                    env.IMAGE_TAG = sh(
                        script: 'git rev-parse --short=12 HEAD',
                        returnStdout: true
                    ).trim()

                    env.FULL_IMAGE =
                        "${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
                }

                sh '''
                    echo "IMAGE_TAG=$IMAGE_TAG"
                    echo "FULL_IMAGE=$FULL_IMAGE"
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build \
                      -t "$FULL_IMAGE" \
                      .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'harbor-jenkins',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$HARBOR_PASSWORD" |
                          docker login "$HARBOR_REGISTRY" \
                            -u "$HARBOR_USER" \
                            --password-stdin

                        docker push "$FULL_IMAGE"

                        docker logout "$HARBOR_REGISTRY"
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    kubectl set image \
                      -f deployment.yaml \
                      nginx="$FULL_IMAGE" \
                      --local \
                      -o yaml \
                    | kubectl apply -f -

                    kubectl apply \
                      -f service.yaml
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    kubectl rollout status \
                      deployment/jenkins-k8s-demo \
                      -n "$K8S_NAMESPACE" \
                      --timeout=120s

                    kubectl get pods \
                      -n "$K8S_NAMESPACE"

                    echo "Running image:"

                    kubectl get deployment \
                      jenkins-k8s-demo \
                      -n "$K8S_NAMESPACE" \
                      -o jsonpath='{.spec.template.spec.containers[0].image}'

                    echo
                '''
            }
        }
    }
}
