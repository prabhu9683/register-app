pipeline {
    agent { label 'jenkins_agent' }
    
    tools {
        jdk 'jdk-21'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "register-application-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "prabhu9683"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }    
    stages {
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/prabhu9683/register-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

         stage("Quality Gate") {
               steps {
                   script {
                        waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                          }
                      }
                   }

          stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                 }
              }
          }    
/*
    // Wipes the workspace safely ONLY after the entire pipeline finishes
    post {
        always {
            cleanWs()
        }
    }
*/ 
}
