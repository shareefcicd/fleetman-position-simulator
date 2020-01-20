pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     ECR_URI = "842970055596.dkr.ecr.us-east-1.amazonaws.com/leg888"
     
     SERVICE_NAME = "fleetman-position-simulator"
     REPOSITORY_TAG="${ECR_URI}:${SERVICE_NAME}${BUILD_ID}"
      
   }

   stages {
      
      stage('Build') {
         tools {
               maven 'maven'
         }
         steps {
            
            sh '''mvn clean package'''
         }
      }

  
       stage('artifactory') {
           steps {  
            rtServer ( 
              id: 'ARTIFACTORY_SERVER',
              url: 'http://3.88.42.130:8040/artifactory',
    
              username: 'admin',
              password: 'password'
    // If you're using Credentials ID:
        //     credentialsId: 'artifactup'
    // If Jenkins is configured to use an http proxy, you can bypass the proxy when using this Artifactory server:
        // bypassProxy: true
    // Configure the connection timeout (in seconds).
    // The default value (if not configured) is 300 seconds:
      //    timeout = 300
          )
              
              rtDownload (
               serverId: 'ARTIFACTORY_SERVER',
               spec: '''{
                 "files": [
                  {
                     "pattern": "artart/",
                     "target": "workspace/target/eetman-position-simulator_master/target/*.jar"
                  }
                ]
              }''',
             buildName: 'build',
             buildNumber: '42'
        )
               
            rtUpload (
             serverId: 'ARTIFACTORY_SERVER',
             spec: '''{
              "files": [
               {
                 "pattern": "artart/",
                 "target": "workspace/target/eetman-position-simulator_master/target/*.jar"
               }
           ]
          }''',
 
        buildName: 'build',
        buildNumber: '42'
     )
    }   
  }
     
      stage('Build and Push Image') {
         steps {
           sh 'sudo docker image build -t ${REPOSITORY_TAG} .'
           sh 'sudo $(aws ecr get-login --no-include-email --region us-east-1)'
           sh 'sudo docker push ${REPOSITORY_TAG}'
         }
      }

      stage('Deploy to Cluster') {
          steps {
                    sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
          }
      }
   }
}
