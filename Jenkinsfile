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
        stage ("Build image") {
            steps {
                script {
                    docker.build registry
                }
            }
        }
        
        stage ("Push to ECR") {
            steps {
                sh "aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 855894003578.dkr.ecr.ap-southeast-1.amazonaws.com"
                sh "855894003578.dkr.ecr.ap-southeast-1.amazonaws.com/my-ecr-repo:latest"
                
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