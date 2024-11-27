pipeline {    
    agent any 

    tools {
      //  jdk 'jdk17'
        maven 'maven3'
          }
     environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-amazon-corretto.x86_64'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}" 
    }

    stages {
        
        stage ('git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/harshalgaikwad25/Mission.git']])
            }
        }
        
        stage ('maven compile and test') {
            steps {
                sh   "mvn compile"
             //   sh   "mvn test"
            }
        }
        stage('File system scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage ('mvn build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage ('Archive Artifacts'){
           steps {
               archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
               }
        }
        
        stage ('Docker build and tag') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker' )
                    {
                        sh   "docker build -t mission ."
                        sh   "docker tag mission  harshalgaikwad/mission:latest"
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                        sh "trivy image --format table -o trivy-image-report.html harshalgaikwad/mission:latest"
                    }
                }
            }
        }        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker') {
                        sh "docker push harshalgaikwad/mission:latest"
                    }
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                script {
                    sh "docker stop my-app || true"
                    sh "docker rm my-app || true"
                }
            }
        }        
        stage ('Deploy app on container') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'docker-hub', toolName: 'docker' )
                    {
                        sh "docker run -d --name my-app  -p 8081:8080 harshalgaikwad/mission:latest"

                    }
                }
            }
        }
        def parallel_build_trigger = {
    // List of builds to trigger
    def builds_to_trigger = [    
	
	"Bar-test/second-job/main"
	
	]
	
	 // Define a map to hold the build steps
    def builds = [:]
	
	    for(i = 0; i < builds_to_trigger.size(); i += 1) {
        def current_build = builds_to_trigger[i]
        builds["${current_build}"] = {
          build wait: false, job: "${current_build}"
        }
    }
	
	    // Trigger all builds in parallel
    parallel build
	
hook.addPostHook(StageName.POST_BUILD, parallel_build_trigger)

}
    }
}
