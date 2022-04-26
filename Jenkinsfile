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
            agent {
                kubernetes {
                    cloud 'kubernetes-hmg2'
                    }
                }
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps{
                sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api/Deployment.yaml'
                sh 'cat ./k8s/api/Deployment.yaml'
                sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                sh 'echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check'
                sh 'install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl'
                
                withCredentials([kubeconfigContent(credentialsId : "$Kubeconfig" ,variable : 'KUBECONFIG')]) {
                    sh '''
                        set +x
                        mkdir ~/.kube
                        echo "$KUBECONFIG" > ~/.kube/config
                '''
          }
          sh 'kubectl cluster-info'

                sh 'kubectl apply -f ./k8s/api/'
                kubernetesDeploy(configs: '**/k8s/mongodb/**', kubeconfigId: 'Kubeconfig')

            }
        }
    }
}