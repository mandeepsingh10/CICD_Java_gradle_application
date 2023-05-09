pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            steps{
                script{ 
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube --info'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        stage("Docker Build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_docker_repo_pass', variable: 'nexus_docker_repo_pass_var')]) {
                            sh '''
                            docker build -t 52.66.172.21:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_docker_repo_pass_var 52.66.172.21:8083
                            docker push 52.66.172.21:8083/springapp:${VERSION}
                            docker rmi 52.66.172.21:8083/springapp:${VERSION}
                            '''
                }
                    
                }
            }
        }
        stage('Identifying the misconfiguration in HELM charts using datree'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=ff434798-2f4e-4adf-a167-41b43740c677']) {
                            sh 'sudo helm datree test myapp/'
                        }
                    }
                }
            }
        }
    }

    post {
	    always {
	        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";

		}

    }
}