/////////////////////////////////////DOCKER HUB ///////////////////////////////////////////
// pipeline {
//     agent any

//     stages {
//         stage ('Build Image') {
//             steps {
//                 script {
//                     dockerapp = docker.build("wallafi/web-live-app:${env.BUILD_ID}", "-f ./src/Dockerfile ./src")
//                 }
//             }
//         }

//         stage ('Push Image') {
//             steps {
//                 script {
//                     docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
//                         dockerapp.push('latest')
//                         dockerapp.push("${env.BUILD_ID}")
//                     }
//                 }
//             }
//         }


//         stage ('Deploy no Kubernetes') {
//             environment {
//                 tag_version = "${env.BUILD_ID}"
//             }
//             steps {
//                 withAWS(credentials: 'jenkins-credential', region: 'us-east-1') {
//                     sh 'aws eks update-kubeconfig --name jenkins'
//                     sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
//                     sh 'kubectl apply -f ./k8s/deployment.yaml'
//                 }

//             }
//         }
//     }
// }

/////////////////////////////////////ECR ///////////////////////////////////////////

pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1' // replace with your AWS region
        ECR_REGISTRY = '931783206580.dkr.ecr.us-east-1.amazonaws.com' // replace with your ECR registry URI
        ECR_REPOSITORY = 'wallafi/web-live-app' // replace with your ECR repository name
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("${ECR_REPOSITORY}:${env.BUILD_ID}", "-f ./src/Dockerfile ./src")
                }
            }
        }
        stage('Login to ECR') {
            steps {
                script {
                    withAWS(credentials: 'jenkins-credential', region: 'us-east-1') {
                        sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                    }
                }
            }
        }
        stage('Tag & Push Image to ECR') {
            steps {
                script {
                    def ecrTag = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${env.BUILD_ID}"
                    dockerapp.push("${env.BUILD_ID}")
                    dockerapp.push("latest")
                }
            }
        }

        stage ('Deploy no Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withAWS(credentials: 'jenkins-credential', region: 'us-east-1') {
                    sh 'aws eks update-kubeconfig --name jenkins'
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'
                }

            }
        }
    }
}
