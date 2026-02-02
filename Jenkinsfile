pipeline {
    agent any

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        DOCKERHUB_USER = "gsd2503"
        IMAGE_NAME     = "maven-java-docker-swarm"
        IMAGE_TAG      = "latest"
        FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gsd2503/devops-practice.git',
                    credentialsId: 'github-tocken'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Verify JAR') {
            steps {
                sh 'java -jar target/*.jar'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $FULL_IMAGE .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Docker-jenkins-PAT',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh 'docker push $FULL_IMAGE'
            }
        }
    }

    post {
        success {
            echo "✅ Image pushed to Docker Hub: $FULL_IMAGE"
        }
        always {
            sh 'docker logout || true'
        }
    }
}
