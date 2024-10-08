pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "654654450756"
        REGION = "ap-south-1"
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com"
        IMAGE_NAME = "ilasingh20/august-ms:august-ms-v.1.${env.BUILD_NUMBER}"
        ECR_IMAGE_NAME = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/august-ms:august-ms-v.1.${env.BUILD_NUMBER}"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
    }

    tools {
        maven 'maven_3.9.4'
        // sonarqubeScanner 'sonarqube-scanner'
    }

    stages {
        stage('Code Compilation') {
            steps {
                echo 'Code Compilation is In Progress!'
                sh 'mvn clean compile'
                echo 'Code Compilation is Completed Successfully!'
            }
        }
        stage('Code Package') {
            steps {
                echo 'Creating WAR Artifact'
                sh 'mvn clean package'
                echo 'Artifact Creation Completed'
            }
        }
        stage('Building & Tag Docker Image') {
            steps {
                echo "Starting Building Docker Image: ${env.IMAGE_NAME}"
                sh "docker build -t ${env.IMAGE_NAME} ."
                echo 'Docker Image Build Completed'
            }
        }
        stage('Docker Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_ILA_CRED', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    echo "Pushing Docker Image to DockerHub: ${env.IMAGE_NAME}"
                    sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    sh "docker push ${env.IMAGE_NAME}"
                    echo "Docker Image Push to DockerHub Completed"
                }
            }
        }
        stage('Docker Image Push to Amazon ECR') {
            steps {
                echo "Tagging Docker Image for ECRq: ${env.ECR_IMAGE_NAME}"
                sh "docker tag ${env.IMAGE_NAME} ${env.ECR_IMAGE_NAME}"
                echo "Docker Image Tagging Completed"

                withDockerRegistry([credentialsId: 'ecr:ap-south-1:ecr-credentials', url: "https://${ECR_URL}"]) {
                    echo "Pushing Docker Image to ECR: ${env.ECR_IMAGE_NAME}"
                    sh "docker push ${env.ECR_IMAGE_NAME}"
                    echo "Docker Image Push to ECR Completed"
                }
            }
        }
        stage('Sonarqube') {
            environment {
                scannerHome = tool 'qube'
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                    sh 'mvn sonar:sonar'
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }

    }
}