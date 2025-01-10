@Library('my_jenkins_shared_library')_

pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID  = 'docker_hub_credentials'      // DockerHub credentials ID
        imageName               = 'mohamedabonazel/ivolve1'      // DockerHub image name
        githubCredentialsID     = 'github_credentials'           // GitHub credentials ID
        kubeconfigCredentialsID = 'my-kubeconfig'                // Kubernetes kubeconfig credentials ID
        kubernetesClusterURL    = 'https://192.168.49.2:8443'    // Kubernetes Cluster Control Plane URL
        KUBECONFIG              = '~/.kube/config' 
        // Certificates as credentials from Jenkins credentials store
        minikubeCACertID        = 'ca-crt'             // CA certificate ID
        minikubeClientCertID    = 'client-crt'         // Client certificate ID
        minikubeClientKeyID     = 'client-key'          // Client key ID
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

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    
                    // Fetch the certificates from Jenkins credentials store
                    withCredentials([
                        file(credentialsId: "${minikubeCACertID}", variable: 'CA_CERT_PATH'),
                        file(credentialsId: "${minikubeClientCertID}", variable: 'CLIENT_CERT_PATH'),
                        file(credentialsId: "${minikubeClientKeyID}", variable: 'CLIENT_KEY_PATH')
                    ]) {
                        deployOnkubernates("${kubeconfigCredentialsID}", "${kubernetesClusterURL}", "${imageName}", readFile(CA_CERT_PATH), readFile(CLIENT_CERT_PATH), readFile(CLIENT_KEY_PATH))
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

