pipeline {
    agent any

    environment {
        COMMIT_ID = ""
    }

    stages {

        stage('Preparation') {
            steps {
                checkout scm

                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    COMMIT_ID = readFile('.git/commit-id').trim()
                }

                echo "Commit ID = ${COMMIT_ID}"
            }
        }

        stage('Image Build') {
            steps {
                echo 'Building image inside Minikube...'

                // Copy source code into Minikube
                sh "scp -i ~/.minikube/machines/minikube/id_rsa -r ./* docker@$(minikube ip):/home/docker/"

                // Build Docker image inside Minikube
                sh "minikube ssh 'docker build -t webapp:${COMMIT_ID} /home/docker/'"
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'

                // Update image in manifest
                sh """
                    sed -i -r 's#richardchesterwood/k8s-fleetman-webapp-angular:release2#webapp:${COMMIT_ID}#' ./manifests/workloads.yaml
                """

                sh 'kubectl apply -f ./manifests/'
                sh 'kubectl get pods'
            }
        }
    }
}
