pipeline {
    agent any

    environment {
        DATE_TAG = "${new Date().format('yyyy-MM-dd')}"
        DOCKER_CREDENTIALS_ID = 'docker-credentials-burak'
    }

    stages {
        stage('SCM') {
            steps {
                checkout scmGit(branches: [[name: '*/dev']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-credentials-burak', url: 'https://github.com/capitansec/vf-assignment-test.git']])
            }
        }

        stage('Build and Push Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-burak') {
                        def imageTag = "capitansec/vf-assignment-test:${env.DATE_TAG}-${env.BUILD_ID}"
                        def dockerImage = docker.build(imageTag)
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
