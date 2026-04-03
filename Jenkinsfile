pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node25'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "mukulshukla030503/zomato-app"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Mukulshukla353/zomato.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=zomato \
                    -Dsonar.projectName=zomato \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                }
            }
        }

        stage('Push Image to Dockerhub') {
            steps {
                sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Update Deployment Image') {
            steps {
                sh '''
                sed -i "s|image: .*|image: $IMAGE_NAME:$BUILD_NUMBER|g" Kubernetes/deployment.yaml
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl apply -f Kubernetes/deployment.yaml
                kubectl apply -f Kubernetes/service.yaml
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build Successful

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                Docker Image: ${IMAGE_NAME}:${BUILD_NUMBER}

                ${env.BUILD_URL}
                """,
                to: "mukulshukla030503@gmail.com"
            )
        }

        failure {
            emailext(
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build Failed

                Job Name: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}

                ${env.BUILD_URL}
                """,
                to: "mukulshukla030503@gmail.com"
            )
        }
    }
}
            }
        }
    }
}
