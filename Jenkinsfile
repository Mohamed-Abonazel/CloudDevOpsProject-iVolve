@Library('my_jenkins_shared_library')_

pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID  = 'docker_hub_credentials'     // DockerHub credentials ID
        imageName               = 'mohamedabonazel/ivolve1:latest' // DockerHub image name
        githubCredentialsID     = 'github_credentials'        // GitHub credentials ID
        kubeconfigCredentialsID = 'kubeconfig_credentials'    // Kubernetes kubeconfig credentials ID
        kubernetesClusterURL    = 'https://192.168.49.2:8443' // Kubernetes Cluster Control Plane URL
        kubernetesNamespace     = 'default'                  // Kubernetes namespace for deployment
    }
    
    stages {
        
        stage('Repo Checkout') {
            steps {
                script {
                    checkoutRepo()
                }
            }
        }

        stage('Run Unit Test') {
            steps {
                script {
                    dir('FinalProjectCode') {
                    // Make gradlew executable
                        sh 'chmod +x ./gradlew'
                        runUnitTests()
                    }
                }
            }
        }
        
        stage('Run SonarQube Analysis') {
            steps {
                script {
                    dir('FinalProjectCode') {
                        runSonarQubeAnalysis()
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    dir('FinalProjectCode') {
                        buildandPushDockerImage("${dockerHubCredentialsID}", "${imageName}")
                    }
                }
            }
        }

        stage('Validate Kubernetes Cluster') {
            steps {
                withKubeConfig([credentialsId: "${kubeconfigCredentialsID}"]) {
                    script {
                        echo "Validating Kubernetes Cluster Connection..."
                        sh '''
                        kubectl cluster-info
                        kubectl get nodes
                        '''
                    }
                }
            }
        }

        stage('Deploy on Kubernetes') {
            steps {
                withKubeConfig([credentialsId: "${kubeconfigCredentialsID}"]) {
                    script {
                        echo "Deploying application on Kubernetes..."
                        dir('Kubernetes') {
                            sh '''
                            kubectl apply -f .
                            kubectl rollout status deployment/ivolve-app -n ${kubernetesNamespace}
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline succeeded"
        }
        failure {
            echo "${JOB_NAME}-${BUILD_NUMBER} pipeline failed"
        }
    }
}

