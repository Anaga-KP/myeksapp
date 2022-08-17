pipeline {
   
    agent any
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

