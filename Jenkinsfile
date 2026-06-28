pipeline {
    agent any

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
                sh 'mvn package -DskipTests'
                // Wildcard safety: copies whatever versioned jar maven creates into a clean app.jar
                sh 'cp target/twitter-app-*.jar target/app.jar'
            }
        }

        stage('Upload to Nexus') {
                    steps {
                        script {
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
                                        type: 'jar',         // Added type explicitly
                                        extension: 'jar', 
                                        classifier: '', 
                                        file: 'target/app.jar'
                                    ]
                                ]
                            )
                        }
                    }
                }
