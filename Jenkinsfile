pipeline {
    agent any

    environment {
        IMAGE_NAME = "erdigvijay/devops_repo:customer-service-${BUILD_NUMBER}"
        K8S_NAMESPACE = "automotive"
        DEPLOYMENT_NAME = "customer-service"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/erdigu/customer-service.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push ${IMAGE_NAME}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                  kubectl apply -f customer-service.yaml
                  kubectl set image deployment/${DEPLOYMENT_NAME} \
                    customer-service=${IMAGE_NAME} \
                    -n ${K8S_NAMESPACE}
                """
            }
        }

        stage('Verify Rollout') {
            steps {
                sh """
                  kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                  kubectl get pods -n ${K8S_NAMESPACE} -l app=customer-service
                """
            }
        }
    }
}
