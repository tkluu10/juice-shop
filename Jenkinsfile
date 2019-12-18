pipeline {
    environment {
        staging_user ="ec2-user"
        staging_ip = "3.226.251.111"
        sonarURL = "http://192.168.1.162:9000/"
        sonarToken = "22a0a7b129d8cceb3dd3c14a327b78672939c763"
        registry = "tkluu10/juice-shop"
        registryCredential = 'dockerhub'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository...'
                git 'https://github.com/tkluu10/juice-shop.git'
            }
        }
        stage('SAST') {
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
                sleep(time: 5, unit: 'SECONDS')
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker Image') {
            steps{
                echo "Building image..."
                script {
                    dockerImage = docker.build("tkluu10/juice-shop:${env.BUILD_ID}")
                }
            }
        }
        stage('Test Docker Image') {
            steps {
                echo 'Testing image...'
            }
        }
        stage('Push Docker Image') {
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
        stage('Deploy to Staging Environment') {
            steps {
                echo "Deploying to staging server..."
                sshagent(credentials : ['staging_login']) {
                    sh 'ssh -o StrictHostKeyChecking=no ${staging_user}@${staging_ip} ./deploy.sh'
                }
            }
        }
        stage('DAST') {
            steps {
                echo "Running ZAP baseline scan..."
                sshagent(credentials : ['staging_login']) {
                    sh 'ssh -o StrictHostKeyChecking=no ${staging_user}@${staging_ip} "docker run -t owasp/zap2docker-stable zap-baseline.py -t ${staging_ip}:3000 || true "'
                }
            }
        }
    }
}