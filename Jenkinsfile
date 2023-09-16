pipeline {
    agent { label 'jenkins-worker' }
    tools {
        jdk 'jdk17'
        maven 'Maven3'
    }
    // environment {
	   //  APP_NAME = "register-app-pipeline"
    //         RELEASE = "1.0.0"
    //         DOCKER_USER = "narian318"
    //         DOCKER_PASS = 'dockerhub'
    //         IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
    //         IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    
    // }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Narian318/register-app.git'
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

       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'sonar-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }

       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }	
            }

        }

       stage ('Build and push to docker hub'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build -t register-app-pipeline ."
                        sh "docker tag register-app-pipeline narian318/register-app-pipeline:latest"
                        sh "docker push narian318/register-app-pipeline:latest"
                   }
                }
            }
        }

       stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image narian318/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi register-app-pipeline:latest"
               }
          }
       }

       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user Narendra N:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-54-224-147-169.compute-1.amazonaws.com:8080/job/gitops-register-app-CD/buildWithParameters?token=gitops-token'"
                }
            }
       }
    }
}
