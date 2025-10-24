pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-login')
        IMAGE_NAME = "tonnom/monapp"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: '*/main', url: 'https://github.com/<ton_nom>/mon-projet-devops.git'
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { env.BRANCH_NAME != 'master' }
            }
            steps {
                sh 'helm upgrade --install myapp ./charts --namespace dev --set image.tag=$BUILD_NUMBER'
            }
        }

        stage('Manual Approval for Production') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Déployer en production ?'
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                sh 'helm upgrade --install myapp ./charts --namespace prod --set image.tag=$BUILD_NUMBER'
            }
        }
    }
}

