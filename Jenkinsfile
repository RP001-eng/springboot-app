pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = 'rp001/springboot-app:v1'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/RP001-eng/springboot-app.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=springboot-app \
                        -Dsonar.projectName=springboot-app \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                        """
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
