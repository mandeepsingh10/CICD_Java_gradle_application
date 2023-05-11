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
                    timeout(time: 15, unit: 'MINUTES') {
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
                    withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
                        sh '''
                        docker build -t 15.206.89.243:8083/springapp:${VERSION} .
                        docker login -u admin -p $nexus_pass_var 15.206.89.243:8083
                        docker push 15.206.89.243:8083/springapp:${VERSION}
                        docker rmi 15.206.89.243:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        }
        stage('Identifying the misconfiguration in HELM charts using datree'){
            steps{
                script{
                    dir('kubernetes/') {
                        withCredentials([string(credentialsId: 'datree-token', variable: 'datree_token_var')]) {
                            sh '''
                            sudo helm datree config set token $datree_token_var
                            sudo helm datree test myapp/
                            '''
                        }    
                    }
                }
            }
        }
        stage("Pushing Helm Charts to Nexus Repo"){
            steps{
                script{
                    dir('kubernetes/'){
                        withCredentials([string(credentialsId: 'nexus_pass', variable: 'nexus_pass_var')]) {
                            sh '''
                            helmversion=$(helm show chart myapp/ | grep version | awk '{print $2}')
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u admin:$nexus_pass_var http://15.206.89.243:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }   
                    }
                }   
            }   
        }
        stage('Deploying application on k8s-cluster') {
            steps {
                script{
                    dir ("kubernetes/"){  
				        sh 'helm list'
				        sh 'helm upgrade --install --set image.repository="15.206.89.243:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
			        }
                }
            }
        }    
    }
    post {
	    always {
	        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "mandeepsingh1018@gmail.com";
            slackSend channel: '#jenkins-bot', message: "Project: ${env.JOB_NAME} \n Build Number: ${env.BUILD_NUMBER} \n Status: *${currentBuild.result}* \n More info at: ${env.BUILD_URL}"
        }    
    }
}

