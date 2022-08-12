pipeline {
    agent any
    tools {
        maven "Maven3"
    }
    environment {
        registry = "651003575209.dkr.ecr.ap-south-1.amazonaws.com/my-docker-repo"
    }

    stages {
        stage("Checkout from SCM") {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/demuduraviteja/springboot-app.git']]])
            }
        }
        stage("Build jar") {
            steps {
                sh "mvn clean install"
            }
        }
        stage('quality analysis'){
            steps{
                withSonarQubeEnv('SonarQube'){
                sh "mvn sonar:sonar"
                }
            }
        }
        stage("Quality Gate status check") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
          stage("uploading artifacts to jfrog"){
            steps{
                script{
                    sh """curl -u ravi:Ravi123@ -X PUT "https://demuravi.jfrog.io/artifactory/ravi/" -T target/springbootApp.jar"""
                }
            }
        }
        stage("Build image") {
            steps {
                script {
                    docker.build registry
                }
            }
        }
        stage("push to ECR") {
            steps {
                script {
                    sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 651003575209.dkr.ecr.ap-south-1.amazonaws.com"
                    sh "docker push 651003575209.dkr.ecr.ap-south-1.amazonaws.com/my-docker-repo:latest"
                }
            }
        }
        stage("K8s Deploy") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'eks-kubeconfigfile', namespace: '', serverUrl: '') {
                sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
            }
        }
    }
}
