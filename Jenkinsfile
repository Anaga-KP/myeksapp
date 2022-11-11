pipeline {
   agent any
   environment {
        registry = "616090633012.dkr.ecr.us-east-1.amazonaws.com/ecr-repo"
    }
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
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry 
        }
      }
    }
        stage('Pushing to ECR') {
            steps{  
               script {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 616090633012.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker push 616090633012.dkr.ecr.us-east-1.amazonaws.com/ecr-repo:latest'
         }
        }
      }
       stage('K8S Deploy') {
      steps{   
            script {
                withKubeConfig([credentialsId: 'k8s', serverUrl: '']) {
                sh ('kubectl apply -f  eks-deployment.yaml')
                }
            }
        }
       }
     
    }
 }
