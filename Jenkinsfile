pipeline {
    environment {
    registry = "Ratikk/spring-petclinic"
    registryCredential = 'dockerhub'
    AWS_ACCOUNT_ID="XXX"
    AWS_DEFAULT_REGION="us-east-2" 
    IMAGE_REPO_NAME="demo"
    REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
   
    
  }
    agent any
    tools {
        maven "MAVEN"
        jdk "JDK"
    }
    stages {
        stage('Initialize'){
            steps{
                echo "PATH = ${M2_HOME}/bin:${PATH}"
                echo "M2_HOME = /opt/apache-maven-3.8.2"
                
            }
        }
        stage('Build') {
            steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                sh './mvnw package'
               
                }
            }
        }  
        stage('Jfrog Upload') {
             steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD/target") {
                sh 'jfrog c add --artifactory-url="https://XXXX.jfrog.io/artifactory/" --user="demo" --password="XXX"  --interactive="false"'
                 sh 'jfrog rt u "spring-petclinic-2.6.0-SNAPSHOT.jar" "test/pring-petclinic-2.6.0-SNAPSHOT-$BUILD_NUMBER.jar" --recursive=false'
               
                }
            }
        }  
            
        stage('Test') {
            steps {
                dir("/var/lib/jenkins/workspace/MAVENBUILD") {
                    sh 'mvn test'
                }
            }
        }
            
        
     stage('Building DockerHub image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Push Image to DockerHub') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
        
    stage('Building ECR image') {
      steps{
        script {
          dockerImageAws = docker.build REPOSITORY_URI + ":$BUILD_NUMBER"
        }
      }
    }
     stage('Deploy to AWS ECR') {
            steps {
                script{
                    docker.withRegistry('https://XXXX.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:awscred') {
                    dockerImageAws.push()
                    
                    }
                }
            }
        }
        
   stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
        sh "docker rmi $REPOSITORY_URI:$BUILD_NUMBER"
        
      }
    }
   stage('Cleanup Working Directory') {
            steps{
                cleanWs deleteDirs: true
            }
        }
  }
}
