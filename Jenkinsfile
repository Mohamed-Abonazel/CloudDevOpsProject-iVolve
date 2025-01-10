@Library('my_jenkins_shared_library')_

pipeline {
    agent any
    
    environment {
        dockerHubCredentialsID  = 'docker_hub_credentials'       // DockerHub credentials ID
        imageName               = 'mohamedabonazel/ivolve1'      // DockerHub image name
        githubCredentialsID     = 'github_credentials'           // GitHub credentials ID
        CD_SERVER_URL = 'http://197.167.39.201:8080'             // Replace with the CD Jenkins server URL
        CD_JOB_NAME = 'ivolve-cd-deploy'                        // Replace with the CD pipeline job name
        CD_TRIGGER_TOKEN = 'TRIGGER_TOKEN'                       // Replace with the trigger token
        CD_USER = 'Mohamed-Abonazel'                                // Replace with CD Jenkins server username
        CD_API_TOKEN = 'API-TOKEN'                          // Replace with the API token
        
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


        stage('Trigger CD Pipeline') {
            steps {
                echo "Triggering CD pipeline on the CD Jenkins server..."
                sh """
                curl -X POST -u ${CD_USER}:${CD_API_TOKEN} \
                ${CD_SERVER_URL}/job/${CD_JOB_NAME}/buildWithParameters?token=${CD_TRIGGER_TOKEN}
                """
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

    post {
        success {
            echo "CI pipeline completed successfully, and CD pipeline triggered."
        }
        failure {
            echo "CI pipeline failed or CD pipeline could not be triggered."
        }
    }
}

