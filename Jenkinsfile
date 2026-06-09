pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = "rp001/springboot-app:${BUILD_NUMBER}"
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

        stage('Update GitOps Repo') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'gitops-creds',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )
                ]) {
                    sh '''
                    rm -rf springboot-gitops

                    git clone https://${GIT_USER}:${GIT_PASS}@github.com/RP001-eng/springboot-gitops.git

                    cd springboot-gitops

                    sed -i "s|image: .*|image: rp001/springboot-app:${BUILD_NUMBER}|g" deployment.yaml

                    git config user.email "jenkins@local.com"
                    git config user.name "Jenkins"

                    git add deployment.yaml

                    git commit -m "Updated image to build ${BUILD_NUMBER}" || true

                    git push origin main
                    '''
                }
            }
        }
    }
}
