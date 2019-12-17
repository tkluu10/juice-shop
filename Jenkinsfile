pipeline {
    environment {
        sonarURL = "http://localhost:9000"
        sonarToken = "b99823005c524b84ec0083c7e9aab7ef58b048d0"
        registry = "tkluu10/juice-shop"
        registryCredential = 'dockerhub'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Clone') {
            steps {
                echo 'Cloning repository...'
                git 'https://github.com/tkluu10/juice-shop.git'
            }
        }
        stage("Code Analysis") {
            agent {
                docker { image 'node:12' }
            }
            steps {
                echo 'Performing static code analysis...'
                sh 'npm install -g sonarqube-scanner'
                sh 'sonar-scanner -Dsonar.host.url=${sonarURL} -Dsonar.login=${sonarToken}'
            }
        }
        stage('Build Image') {
            steps{
                echo "Building image..."
                script {
                    dockerImage = docker.build("tkluu10/juice-shop:${env.BUILD_ID}")
                }
            }
        }
        stage('Test Image') {
            steps {
                echo 'Testing image...'
            }
        }
        stage('Push Image') {
            steps {
                echo 'Pushing image to DockerHub...'
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Deploy Image') {
            steps {
                echo "Deploying to staging server..."
                sshagent(credentials : ['staging_server']) {
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@$staging_ip ./deploy.sh'
                }
            }
        }
    }
}