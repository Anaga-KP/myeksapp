pipeline {
   tools {
        maven 'Maven3'
    }
    agent any
    environment {     
           
            registryCredential = 'nexus'
            registry = "172.31.30.165:8083/" 
            NEXUS_VERSION = "nexus3"
            NEXUS_PROTOCOL = "http"
            NEXUS_URL = "172.31.30.165:8081"
            NEXUS_REPOSITORY = "my-docker-images"
            NEXUS_CREDENTIAL_ID = "nexus"
    } 
    stages {
        stage('Cloning Git') {
            steps {
                    checkout([$class: 'GitSCM', 
                              branches: [
                                            [name: '*/main']
                                        ],
                              extensions: [], 
                              userRemoteConfigs: [
                                                    [credentialsId: 'git', 
                                                     url: 'https://github.com/AbdulShukur007/myeksapp.git'
                                                    ]
                                                 ]
                             ])
            }
        }
        stage('SonarQube analysis'){
              steps{
                     script{
                         withSonarQubeEnv('sonarserver') { 
                         sh "mvn sonar:sonar"
                              }
                         timeout(time: 2, unit: 'MINUTES') {
                              waitForQualityGate abortPipeline: true
                              }
                         
                      }
               }
        }
        stage('Build') {
               steps {
                       sh 'npm install'
                       sh '<<Build Command>>'
                }
        }  
     
        stage ("terraform init") {
            steps {
                sh ('terraform init') 
            }
        }
        
        stage ("terraform Action") {
            steps {
                echo "Terraform action is --> ${action}"
                sh ('terraform ${action} --auto-approve') 
            }
         }
      
 }

}
