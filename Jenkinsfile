pipeline {
    agent { label 'jenkins_agent' }
    
    tools {
        jdk 'jdk-21'
        maven 'Maven3'
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
    }

    // Wipes the workspace safely ONLY after the entire pipeline finishes
    post {
        always {
            cleanWs()
        }
    }
}
