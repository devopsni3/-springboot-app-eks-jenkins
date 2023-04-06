pipeline {
    agent any

    environment {
        registry = "855894003578.dkr.ecr.ap-southeast-1.amazonaws.com/my-ecr-repo"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/devopsni3/springboot-app-eks-jenkins.git']])
            }
        }
        
        stage ("build Jar") {
            steps {
                sh "mvn clean install"
            }
        }
        stage('SonarQube analysis') {
          steps {
            withSonarQubeEnv('sonar') {
              sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=devopsni3'
              }
            }
          }          
    // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":prod-$BUILD_NUMBER"
                    }
                }
            }
            // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{  
                script {
                    docker.withRegistry('https://855894003578.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:aws_cred') {
                        dockerImage.push()
                        }
                    }
                }
            }
        stage('Remove Unused Dockerimage') {
            steps{
                sh "docker rmi $registry:prod-$BUILD_NUMBER"
           }
        }        
        stage ("Deploy to K8S") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                sh "kubectl apply -f eks-deploy-k8s.yaml"
                    
                }
            }
        }
    }
}