pipeline {
    agent any

    stages {

        stage('Checkout  Repositoro') {
            steps {
                git url: 'https://github.com/acthiago/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Docker Build Image') {
            steps {
                script {
                    dockerapp = docker.build("acthiago/api-pedelogo:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps{
                agent {
                    kubernetes {
                        cloud 'kubernetes-hmg2'
                    }
                }
                environment {
                    tag_version = "${env.BUILD_ID}"
            
                }
            }
        }
    }
}