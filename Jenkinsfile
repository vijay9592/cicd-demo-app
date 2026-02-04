pipeline {
    agent any

    environment {
        IMAGE_NAME = "dockerhubusername/cicd-demo"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/cicd-demo-app.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                pip install -r requirements.txt
                pytest
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=cicd-demo \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    docker login -u $USER -p $PASS
                    docker build -t $IMAGE_NAME .
                    docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker rm -f cicd-demo || true
                docker run -d --name cicd-demo $IMAGE_NAME
                '''
            }
        }
    }
}
