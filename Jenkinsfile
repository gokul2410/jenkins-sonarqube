pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "gokul2410/python-ci-cd"
    }

    stages {

        stage('Install & Test') {
            steps {
                sh '''
                pip3 install --break-system-packages -r requirements.txt
                python3 -m pytest
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    script {
                        def scannerHome = tool 'sonar-scanner'
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=python-ci-cd \
                        -Dsonar.sources=. \
                        -Dsonar.python.version=3
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', 
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker build -t $DOCKER_IMAGE .
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}
