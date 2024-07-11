pipeline {
    agent any

    environment {
        DATE_TAG = "${new Date().format('yyyy-MM-dd')}"
        DOCKER_CREDENTIALS_ID = 'docker-credentials-burak'
        KUBERNETES_CREDENTIALS_ID = 'kubernetes-credentials-burak'
        NAMESPACE = 'vf-assesment'
    }

    stages {
        stage('SCM') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-credentials-burak', url: 'https://github.com/capitansec/vf-assignment-test.git']])
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

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([kubeconfigFile(credentialsId: env.KUBERNETES_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
                        def imageTag = "capitansec/vf-assignment-test:${env.DATE_TAG}-${env.BUILD_ID}"
                        sh "kubectl set image deployment/vf-assesment-deployment spring-boot-application=${imageTag} --kubeconfig=${KUBECONFIG} --namespace=${env.NAMESPACE}"
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'cadb0245-e2dc-41b3-b75b-433a223', gitToolName: 'Default')]) {
                    sh sh "sed -i 's|image: capitansec/vf-assignment-test:.*|image: ${imageTag}|g' k8s/deployment.yaml"
                    sh 'git add k8s/deployment.yaml'
                    sh 'git commit -m 'Update deployment image to ${imageTag}'
                    sh 'git push origin main'
                    sh 'rm -rf *'
                }
            }
        }
    }
}