pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '980268627925'
        AWS_REGION     = 'us-east-1'
        ECR_REPO_NAME  = 'blogging-app-frontend'
        ECR_URI        = "980268627925.dkr.ecr.us-east-1.amazonaws.com/blogging-app-frontend"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    tools {
        maven 'MAVEN_HOME'
        jdk 'JAVA_HOME'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Kurdeep/FullStack-Blogging-App'
            }
        }

        stage('Compile & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Package Artifact') {
            steps {
                // Just let Maven build the clean versioned jar file alone
                sh 'mvn package -DskipTests'
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // Dynamically grab the exact jar path for the Nexus uploader stage
                    def jarFile = findFiles(glob: 'target/twitter-app-*.jar')[0]
                    
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '32.199.170.32:8081',
                        groupId: 'com.example', 
                        version: '1.0.0', 
                        repository: 'blogging-app-repo',
                        credentialsId: 'nexus-credentials', 
                        artifacts: [
                            [
                                artifactId: 'twitter-app', 
                                type: 'jar',
                                extension: 'jar', 
                                classifier: '', 
                                file: "${jarFile.path}"
                            ]
                        ]
                    )
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                sh "docker build -t ${ECR_REPO_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}"
                sh "docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_URI}:latest"
            }
        }

        stage('Push to AWS ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}"
                sh "docker push ${ECR_URI}:${IMAGE_TAG}"
                sh "docker push ${ECR_URI}:latest"
            }
        }
    }
}
