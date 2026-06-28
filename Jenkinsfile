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
                // This command triggers the creation of your JAR file
                sh 'mvn package -DskipTests'
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
                        version: '0.0.1-SNAPSHOT', // Matches standard spring boot starter defaults
                        repository: 'blogging-app-repo',
                        credentialsId: '', 
                        artifacts: [
                            [artifactId: 'twitter-app', ext: 'jar', classifier: '', file: 'target/twitter-app-0.0.1-SNAPSHOT.jar']
                        ]
                    )
                }
            }
        }
    }
}
