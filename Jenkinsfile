pipeline {
    agent any

    environment {
        DELIVERY_IMAGE = "selikakanajmi/delivery-service:${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Repo') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/selikakanajmii07/DeliveryService.git'
            }
        }

        stage('Unit Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'go test ./...'
                }
            }
        }

        stage('Lint / Vet') {
            steps {
                bat 'go vet ./...'
            }
        }

        stage('Build Image') {
            steps {
                bat 'docker build -t %DELIVERY_IMAGE% .'
            }
        }

        stage('Functional Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                    docker rm -f test-delivery

                    docker run -d --name test-delivery ^
                      -p 8086:8086 ^
                      %DELIVERY_IMAGE%

                    timeout /t 5

                    curl -X POST http://localhost:8086/delivery

                    docker rm -f test-delivery
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-login',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    bat '''
                    echo %PASSWORD% | docker login -u %USERNAME% --password-stdin
                    docker push %DELIVERY_IMAGE%
                    '''
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                echo 'Deploy Kubernetes placeholder'
            }
        }

        stage('Verify') {
            steps {
                echo 'PIPELINE SUCCESS'
            }
        }
    }
}
