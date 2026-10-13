pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'eyakochti'
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                dir('backend') {
                    sh 'mvn clean compile'
                }
            }
        }

        stage('Tests') {
            steps {
                dir('backend') {
                    sh 'mvn test'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                            mvn sonar:sonar \
                              -Dsonar.projectKey=gestion-projets \
                              -Dsonar.projectName=GestionProjets
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t $DOCKERHUB_USER/gestion-projets-backend:$BUILD_NUMBER ./backend
                    docker build -t $DOCKERHUB_USER/gestion-projets-frontend:$BUILD_NUMBER ./frontend
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push $DOCKERHUB_USER/gestion-projets-backend:$BUILD_NUMBER
                        docker push $DOCKERHUB_USER/gestion-projets-frontend:$BUILD_NUMBER

                        docker logout
                    '''
                }
            }
        }
    }
}
