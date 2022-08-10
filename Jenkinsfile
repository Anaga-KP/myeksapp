pipeline {
   tools {
        maven 'Maven3'
    }
    
    agent any
  //  environment {     
            imagename = "abdulsukku/docker-new"
            registryCredential = 'dockerpass'
            dockerImage = ''    
    } 
   
    stages {
        stage('Cloning Git') {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'githubcred', url: 'https://github.com/AbdulShukur007/myeksapp.git']]])
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
        stage('Maven Build'){
            steps {
                sh 'mvn clean install'           
            }
        }
        stage('Push to Nexus'){
           steps { 
              nexusArtifactUploader artifacts: [
                             [artifactId: 'springbootApp',
                              classifier: '', 
                              file: 'target/MyAwesomeApp-0.0.1.jar', 
                              type: 'jar']
              ], 
                 
              credentialsId: 'nexus', 
              groupId: 'com.tcs.angularjs', 
              nexusUrl: '172.31.9.174', 
              nexusVersion: 'nexus3', 
              protocol: 'http', 
              repository: 'http://44.202.188.182:8081/repository/myeksapp/', version: '0.0.1'
           }
        }
//        stage("Docker build"){
            steps{
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
   //     stage("Push Image to Docker Hub"){
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                    }
                }
            }
        }
      //  stage("kubernetes deployment"){
            steps{
                script {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8s', namespace: '', serverUrl: '') {
                        sh ('kubectl apply -f eks-deployment.yaml')
                    }
                } 
            }
        }
      
    }
}
