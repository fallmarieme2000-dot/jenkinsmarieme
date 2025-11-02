pipeline {
    agent any

    tools {
        nodejs "NodeJS_20"
    }

    environment {
        DOCKER_HUB_USER = 'marieme0516'
        FRONT_IMAGE = 'react-frontend'
        BACK_IMAGE  = 'express-backend'
        AWS_REGION = 'us-west-2'
        TF_DIR = './terraform'
    }

    stages {
        stage('Terraform Init') {
            steps {
                git branch: 'main', url: 'https://github.com/fallmarieme2000-dot/terraform.git'
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-key'],
                    string(credentialsId: 'aws-token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    dir("${TF_DIR}") {
                        sh 'terraform init -upgrade'
                    }
                }
            }
        }

        stage('Terraform Plan & Apply') {
            steps {
                input message: 'Souhaitez-vous appliquer le plan Terraform ?'
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-key'],
                    string(credentialsId: 'aws-token', variable: 'AWS_SESSION_TOKEN')
                ]) {
                    dir("${TF_DIR}") {
                        sh 'terraform plan -out=tfplan'
                        sh 'terraform apply -auto-approve tfplan'
                    }
                }
            }
        }

        stage('Checkout Application') {
            steps {
                git branch: 'main', url: 'https://github.com/fallmarieme2000-dot/jenkinsmarieme.git'
            }
        }

        stage('Install Dependencies') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('backend') {
                            sh 'npm install'
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('frontend') {
                            sh 'npm install'
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t $DOCKER_HUB_USER/$FRONT_IMAGE:latest ./frontend"
                    sh "docker build -t $DOCKER_HUB_USER/$BACK_IMAGE:latest ./backend"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_USER/react-frontend:latest
                        docker push $DOCKER_USER/express-backend:latest
                    '''
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker --version'
                sh 'docker-compose --version || echo "docker-compose non trouvé"'
                sh '''
                    docker-compose -f compose.yaml down || true
                    docker-compose -f compose.yaml pull
                    docker-compose -f compose.yaml up -d
                    docker-compose -f compose.yaml ps
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a réussi ! Détails : ${env.BUILD_URL}",
                to: "fallmarieme1605@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ ÉCHEC: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué. Détails : ${env.BUILD_URL}",
                to: "fallmarieme1605@gmail.com"
            )
        }
    }
}
