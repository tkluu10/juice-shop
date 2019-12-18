pipeline {
    environment {
        stage_server = "3.230.148.253"
        sonarURL = "http://sheikah-server:9000/"
        sonarToken = "bea4127924e6310b3461f38aa7472e19e78e77b1"
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
        stage('Code Analysis') {
            agent {
                docker { image 'node:12' }
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    echo 'Performing static code analysis...'
                    sh 'npm install -g sonarqube-scanner'
                    sh 'sonar-scanner -Dsonar.host.url=${sonarURL} -Dsonar.login=${sonarToken}'
                }
            }
        }
        stage('Quality Gate') {
            steps {
                echo 'Checking Quality Gate...'
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
/*         stage('Build Image') {
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
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@${stage_server} ./deploy.sh'
                }
            }
        } */
    }
}