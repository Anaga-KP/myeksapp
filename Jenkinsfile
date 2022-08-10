pipeline {
   tools {
        maven 'Maven3'
    }
    
    agent any
    environment {     
            imagename = "abdulsukku/docker-new"
            registryCredential = 'dockerpass'
            dockerImage = ''    
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
                                                    [credentialsId: 'githubcred', 
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
        stage('Maven Build'){
            steps {
                sh 'mvn clean install'           
            }
        }
        stage('Push to Nexus'){
           steps { 
              nexusArtifactUploader artifacts: [
                             [artifactId: '${POM_ARTIFACTID}',
                              classifier: '', 
                              file: '${POM_ARTIFACTID}-${POM_VERSION}.${POM_PACKAGING}', 
                              type: '${POM_PACKAGING}']
              ], 
                 
              credentialsId: 'nexus', 
              groupId: '${POM_GROUPID}', 
              nexusUrl: '172.31.9.174:8081', 
              nexusVersion: 'nexus3', 
              protocol: 'http', 
              repository: 'myeksapp', 
              version: '${POM_VERSION}'
           }
        }
        stage("Docker build"){
            steps{
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
        stage("Push Image to Docker Hub"){
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                    }
                }
            }
        }
        stage("kubernetes deployment"){
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
