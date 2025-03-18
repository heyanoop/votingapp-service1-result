pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "heyanoop/resultapp:${BUILD_NUMBER}"
        KUBE_CREDENTIALS = 'cluster-token'
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/heyanoop/votingapp-service1-result.git'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-password', variable: 'DOCKER_PASS')]) {
                    sh """
                    docker login -u heyanoop -p ${DOCKER_PASS}
                    docker build -t ${DOCKER_IMAGE} .
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKER_IMAGE}"
                sh "trivy image --format table ${DOCKER_IMAGE} | tee trivy_scan.log"

                script {
                    echo "🔍 Security Scan Results:"
                    sh "cat trivy_scan.log"
                }
            }
        }

      
        stage('Update Manifest') {
            steps {
               sh "sed -i 's|image: heyanoop/resultapp:.*|image: heyanoop/resultapp:${BUILD_NUMBER}|' k8s-specifications/result-deployment.yaml"
            }
        }


        stage("test"){
            steps{
                sh "cat k8s-specifications/result-deployment.yaml"
            }
        }
 
        stage('Deploy to AKS') {
            steps {
                withKubeConfig([credentialsId: KUBE_CREDENTIALS]) {
                    sh 'kubectl apply -f deployment.yaml -n votingapp'
                }
            }
        }
        }
    }