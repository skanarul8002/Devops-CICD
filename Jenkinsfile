pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK11'
    }

    environment {
        DOCKER_IMAGE = 'quiz-app:latest'
        REGISTRY = 'your-docker-registry'
    }

    stages {
        stage('Tool Install') {
            steps {
                script {
                    // This stage is to ensure the necessary tools are installed
                    echo "Tools are installed"
                }
            }
        }

        stage('Git Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/skanarul8002/quizapp.git']]])
            }
        }

        stage('Code Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --exit-code 1 ${DOCKER_IMAGE}'
            }
        }

        stage('Code Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("${REGISTRY}/${DOCKER_IMAGE}")
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('', 'docker-credentials-id') {
                        docker.image("${REGISTRY}/${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy to Kubernetes using kubectl
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
