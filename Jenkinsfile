pipeline {
  environment {
     DOCKER_REGISTRY_CREDENTIALS = 'docker_hub'
  }        
  agent any
  tools {
    maven 'maven-3.9.3'
  }
  stages {
    stage('cloning our git') {
        steps {
           git branch: 'main', url: 'https://github.com/Nithin93981/mav_sonar_nexus_cicd.git'
        }
    }
    stage('maven unit test and build') {
        steps {
           sh 'mvn test'
           sh 'mvn clean install'
        }
    }
    stage('static code analysis') {
        steps {
            script {
              withSonarQubeEnv(credentialsId: 'sonar_token') {
               sh 'mvn clean package sonar:sonar'
              }
            }  
          
        }

    }
    stage('quality gate status') {
       steps {
         script {
            waitForQualityGate abortPipeline: false, credentialsId: 'sonar_token'
         }
       }
    }
    stage("nexus artifate uploader"){
        steps{
          script{
            def readPomVersion = readMavenPom file: 'pom.xml'  
            // def nexusRepo = readPomVersion.version.endswith("SNAPSHOT") ? "demoapp-snapshot":"demoapp-release "
            nexusArtifactUploader artifacts: [
               [  
                      artifactId: 'devops-integration', 
                      classifier: '', 
                      file: 'target/devops-integration.jar', 
                      type: 'jar'
               ]
            ], 
            credentialsId: 'nexus_auth', 
            groupId: 'com.javatechie', 
            nexusUrl: '54.211.9.74:8081', 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'demoapp-release', 
            // repository: 'nexusRepo'  
            version: "${readPomVersion.version}"
          }  
        }
    }
    stage('docker image build'){
        steps{
            script{
                withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDENTIALS}", passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh  "echo $password | docker login --username $username --password-stdin"
                        sh   "docker build -t  jenkins_test:v$BUILD_NUMBER ."
                        sh   "docker tag  jenkins_test:v$BUILD_NUMBER  suramnithin/jenkins_test:v$BUILD_NUMBER"
                        sh    "docker push  suramnithin/jenkins_test:v$BUILD_NUMBER"
                      
                }
            }
        }
    }
  }
} 
