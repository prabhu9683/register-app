pipeline {
    agent { label 'jenkins_agent' }
    tools {
        jdk 'jdk-21'
        maven 'Maven3'
    }
stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/prabhu9683/register-app'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

        statge("SonarQube Analysis"){
            steps {
                 scripts {
                        withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                        }
                     }
                  }
               }
