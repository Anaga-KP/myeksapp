pipeline {
   tools {
        maven 'Maven3'
    }
    agent any
    environment {     
            imagename = "abdulsukku/docker-new"
            registryCredential = 'nexus'
            registry = "172.31.30.165:8083/"
            dockerImage = ''  
            NEXUS_VERSION = "nexus3"
            NEXUS_PROTOCOL = "http"
            NEXUS_URL = "172.31.30.165:8081"
            NEXUS_REPOSITORY = "my-maven-release"
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
                         withSonarQubeEnv('sonar') { 
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
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
               }  
           }  
        }
        stage("Docker build"){
            steps{
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
        stage("Push Image to Nexus Hub"){
            steps{
                script {
                    docker.withRegistry( 'http://'+registry, registryCredential ) {
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

