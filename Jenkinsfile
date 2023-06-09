pipeline {
  environment {
    imagename = "devops"
    ecrurl = "https://837771900128.dkr.ecr.us-east-1.amazonaws.com"
    ecrcredentials = "ecr:us-east-1:aws-creds"

  } 
  agent any
  stages {
    stage('Cloning Git') {
      steps {
              checkout scm
              sh "whoami "

      }
    }
    stage('Building image') {
      steps{
        script {
           dir("hackathon-starter"){
          dockerImage = docker.build imagename
           }
        }
      }
    }
   
stage('Deploy Master Image') {
      steps{
        script {
          docker.withRegistry(ecrurl, ecrcredentials) {     
            dockerImage.push("$BUILD_NUMBER")
             dockerImage.push('latest')

          }
        }
      }
    }

stage('Trivy Scan') {
            steps {
                script {
                    sh """trivy image --exit-code 0 --severity CRITICAL --scanners vuln 837771900128.dkr.ecr.us-east-1.amazonaws.com/${imagename}:${BUILD_NUMBER} """
                    
                }
                
            }
        }

stage('k8s') {
            steps {
                script {
		  dir("helm")
			{
			    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {
                    sh ('aws eks update-kubeconfig --name basic-cluster --region us-east-1')
			    sh "cd kaiburr && helm upgrade -i kaiburr . --set image.tag=${BUILD_NUMBER} --set mongodburl=mongodb://54.221.132.34:27017/test"
                }
			}	}
                    
                }
                
            }
        }	  

 

  }

  post {
     always {
	  
       cleanWs()
         
       }
    }  
}
