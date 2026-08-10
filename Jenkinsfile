pipeline {

    agent any

    environment {
        IMAGE_NAME = 'jenkins-k8s-demo'
        IMAGE_TAG  = "build-${BUILD_NUMBER}"

        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Build Image') {

            steps {

                sh '''
                    docker build \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} \
                      -t ${IMAGE_NAME}:latest \
                      .
                '''

            }

        }

        stage('Deploy') {

            steps {

                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl set image \
                      deployment/jenkins-k8s-demo \
                      nginx=${IMAGE_NAME}:${IMAGE_TAG} \
                      -n production
                '''

            }

        }

        stage('Verify') {

            steps {

                sh '''
                    kubectl rollout status \
                      deployment/jenkins-k8s-demo \
                      -n production \
                      --timeout=120s

                    kubectl get pods \
                      -n production \
                      -l app=jenkins-k8s-demo
                '''

            }

        }

    }

}
