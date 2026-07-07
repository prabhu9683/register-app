pipeline { 
    agent { 
        label 'jenkins_agent' 
    } 
    tools { 
        jdk 'jdk-21' 
        maven 'Maven3' 
    } 
    environment { 
        app_name = "register-application-pipeline" 
        release = "1.0.0" 
        docker_user = "prabhuv2214" 
        // Best Practice: Use credentials helper instead of hardcoded strings
        docker_credentials_id = 'dockerhub-token' 
        image_name = "${docker_user}/${app_name}" 
        image_tag = "${release}-${BUILD_NUMBER}" 
        // jenkins_api_token = credentials("jenkins_api_token") 
    } 
    stages {
        stage ("Cleanup Workspace") {
           steps {
            // Wipes the workspace safely only after the entire pipeline finishes
            cleanWs()
           }
        }
    
        stage("checkout from scm") { 
            steps { 
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/prabhu9683/register-app' 
            } 
        } 
        
        stage("build application") { 
            steps { 
                sh "mvn clean package" 
            } 
        } 
        
        stage("test application") { 
            steps { 
                sh "mvn test" 
            } 
        }
        
        stage("sonarqube analysis") { 
            steps { 
                script { 
                    withSonarQubeEnv(installationName: 'sonarqube-server', credentialsId: 'jenkins-sonarqube-token') { 
                        sh 'mvn sonar:sonar' 
                    } 
                } 
            } 
        } 
        
        stage("quality gate") { 
            steps { 
                script { 
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token' 
                } 
            } 
        }
        
        stage("build & push docker image") { 
            steps { 
                script { 
                    // Uses registry credentials cleanly via Jenkins Credentials ID
                    docker.withRegistry('', docker_credentials_id) { 
               sh """
                sudo docker build -t ${image_name}:${image_tag} .

                sudo docker tag ${image_name}:${image_tag} ${image_name}:latest

                sudo docker push ${image_name}:${image_tag}

                sudo docker push ${image_name}:latest
                """
                    } 
                } 
            } 
        } 
		/*		
         stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run --user root -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image prabhuv2214/register-application-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
	 */  
		stage ('Cleanup Artifacts') {
           steps {
               script {
				   sh ""
                   sudo docker rmi ${IMAGE_NAME}:${IMAGE_TAG} .
                   sudo docker rmi ${IMAGE_NAME}:latest
				   "" 
               }
          }
       }
   }
}
	
   
    
