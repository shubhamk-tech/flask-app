pipeline {
    agent any
    environment {
        APP_NAME = "flask-app"
        RELEASE = "1.0.0"
        DOCKER_USER = "shubhz1452"
        DOCKER_PASS = "dockerhub-cred"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }


    stages {
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/shubhamk-tech/flask-app.git'
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('k8s-manifests') {
                        kubeconfig(credentialsId: 'k8s', serverUrl: 'https://673008F49C67ADF00C8BD4BFFEAB8CF4.gr7.ap-south-1.eks.amazonaws.com') {
                            sh 'kubectl apply -f deployment.yaml'
                            sh 'kubectl apply -f service.yaml'
                        }        
                    }
                }
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                    "Build Number: ${env.BUILD_NUMBER}<br/>" +
                     "URL: ${env.BUILD_URL}<br/>",
                to: 'shubhkal01@gmail.com',                              
        }
    }
}
