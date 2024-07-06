/////////////////////////////////////DOCKER HUB ///////////////////////////////////////////
pipeline {
    agent any

    stages {
        stage ('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("wallafi/web-live-app:${env.BUILD_ID}", "-f ./src/Dockerfile ./src")
                }
            }
        }

        stage ('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Install AWS CLI') {
            steps {
                sh 'sudo apt-get update && sudo apt-get install -y unzip'
                sh 'curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"'
                sh 'unzip awscliv2.zip'
                sh './aws/install'
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

/////////////////////////////////////ECR ///////////////////////////////////////////

// pipeline {
//     agent any

//     environment {
//         AWS_REGION = 'us-west-2' // replace with your AWS region
//         ECR_REGISTRY = '123456789012.dkr.ecr.us-west-2.amazonaws.com' // replace with your ECR registry URI
//         ECR_REPOSITORY = 'wallafi/web-live-app' // replace with your ECR repository name
//     }

//     stages {
//         stage('Build Image') {
//             steps {
//                 script {
//                     dockerapp = docker.build("${ECR_REPOSITORY}:${env.BUILD_ID}", "-f ./src/Dockerfile ./src")
//                 }
//             }
//         }
//         stage('Login to ECR') {
//             steps {
//                 script {
//                     def loginCommand = sh(script: "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}", returnStdout: true).trim()
//                     sh loginCommand
//                 }
//             }
//         }
//         stage('Tag & Push Image to ECR') {
//             steps {
//                 script {
//                     def ecrTag = "${ECR_REGISTRY}/${ECR_REPOSITORY}:${env.BUILD_ID}"
//                     dockerapp.push("${env.BUILD_ID}")
//                     dockerapp.push("latest")
//                 }
//             }
//         }
//     }
// }
