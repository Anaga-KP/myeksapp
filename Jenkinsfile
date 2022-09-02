pipeline {
   agent any
   tools {
        maven 'Maven3'
    }
    
    stages {
        stage('Cloning Git') {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/Anaga-KP/myeksapp.git']]])
            }
        }  
        stage('Maven Build'){
            steps {
                sh 'mvn clean install'           
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
    
     
    }
 }
