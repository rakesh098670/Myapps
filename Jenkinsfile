pipeline{
  agent any
  tools {
      git 'git'
      maven 'maven'
}

  environment {
        IMAGE_TAG = "${env.BUILD_NUMBER}"
		
              }

stages
{

   stage('SCM Checkout'){
     steps{
            git credentialsId: '342f1cb3-05f8-454f-94ed-7962af5a3c6f', url: 'https://github.com/rakesh098670/Myapps.git'
          }
                        }
						
   stage('Unit Testing'){
     steps{
             sh label: '', script: 'mvn clean test'
           }
                        }
						
   stage ('SonarQube Analysis'){
     steps{
	 
	        script {
                        try {
                            sh 'sudo docker start sonarqube'
							echo "*********wait untill sonarqube starts******************"
							sleep time: 30000, unit: 'MILLISECONDS'
                        } catch (err) {
                            echo 'Docker sonarqube container is already started and Running'
                        }
                    
            withSonarQubeEnv('sonar') {
            withCredentials([string(credentialsId: '5dd773a2-6699-4fd8-adee-2bd5274d8e7c', variable: 'sonar')]) {
            sh 'mvn sonar:sonar -Dsonar.login=${sonar} -Dsonar.password=${sonar}'
                                                                                                                    }

			                          }
                    }
           }
                               }

   stage('Maven packing'){
     steps{
           sh label: '', script: 'mvn clean package'
          }
                          }
   stage ('Binary Management'){
     steps{
           script{
		   
		           try {
                            sh 'sudo docker start nexus'
							echo "*********wait untill nexus starts******************"
							sleep time: 30000, unit: 'MILLISECONDS'
                        } catch (err) {
                            echo 'Docker nexus container is already started and Running'
                        }
				   withCredentials([string(credentialsId: '5dd773a2-6699-4fd8-adee-2bd5274d8e7c', variable: 'nexus')]) {	
				   
                   sh '''curl -v --user ${nexus}:${nexus} --upload-file "target/myweb-0.0.7-SNAPSHOT.war" http://40.117.130.178:8081/repository/myapp/myweb-${IMAGE_TAG}/myweb.war

                    echo "**************** Pull war file from nexus repository********************************"

                    curl -L "http://40.117.130.178:8081/repository/myapp/myweb-${IMAGE_TAG}/myweb.war" -o ${WORKSPACE}/myweb.war -u ${nexus}:${nexus}'''
                                                                                                                       }

                 }



            }
                             }
   stage('Build Docker Image'){
     steps{

  //****** withcredentials is used to encript the docker password **********************
  
           withCredentials([string(credentialsId: '47b34b52-1b0b-4550-a696-0e81dc68d386', variable: 'dockerPSWD')]) {
           sh '''sudo docker login -u rakesh09867 -p ${dockerPSWD}
	       sudo docker build -t rakesh09867/my-app:${IMAGE_TAG} .
	       sudo docker push rakesh09867/my-app:${IMAGE_TAG} '''
                                                                                                                    }
	 
    
	       }
	         post {
                always {
                    script {
                        try {
                            sh 'sudo docker rmi -f $(sudo docker images rakesh09867/my-app -aq)'
                        } catch (err) {
                            echo 'Issues cleaning up images'
                        }
                    }
                }
            }
                              }
    stage('Deploy to dev'){      
    steps {
	
   //************* withcredentials is used to encript the kubeconfig file 	*********************
	
   withCredentials([file(credentialsId: 'aed298cd-40c2-4eaf-9a71-0d25b8cf47b5', variable: 'kubeadmfile')]) {
   sh ''' kubectl get pods -n demo  --kubeconfig  ${kubeadmfile}
   kubectl set image deployment/demodeployment myapp=rakesh09867/my-app:${IMAGE_TAG} -n demo --kubeconfig  ${kubeadmfile}
   kubectl rollout status deployment/demodeployment -n demo --kubeconfig  ${kubeadmfile}'''
        }
	
	    }
                        }
   stage('Input') {
            steps {
                input('Do you want to proceed to dev?')
            }			
			
    }       
 
    stage('Deploy to stagging'){      
      steps {
	
   //******************** withcredentials is used to encript the kubeconfig file *******************
	
   withCredentials([file(credentialsId: '1c709eb0-324f-4c5e-8ef6-68e9e730da66', variable: 'stagging')])  {
   sh ''' kubectl get pods -n demo  --kubeconfig  ${stagging}
   kubectl set image deployment/my-first-deploy mydemoapp=rakesh09867/my-app:${IMAGE_TAG} -n demo --kubeconfig  ${stagging}
   kubectl rollout status deployment/my-first-deploy -n demo --kubeconfig  ${stagging}'''
                                                                                                         }
	
	        }
                              }

}
}
				
	
