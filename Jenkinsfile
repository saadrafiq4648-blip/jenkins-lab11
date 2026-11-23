pipeline {
    agent any
    environment {
        IMAGE_NAME   = 'lab11-site'
        STAGING_PORT = '8090'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -l'
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            environment {
                HOME = '.'
            }
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Package') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }
        stage('Start Staging') {
            steps {
                sh '''
                    docker rm -f lab11-staging || true
                    docker run -d --name lab11-staging -p ${STAGING_PORT}:80 ${IMAGE_NAME}:${BUILD_NUMBER}
                    sleep 3
                    curl -sf http://localhost:${STAGING_PORT}/ > /dev/null
                '''
            }
        }
        stage('Test') {
            agent {
                dockerfile {
                    dir 'tests'
                    args '--network host'
                    reuseNode true
                }
            }
            environment {
                HOME     = '/tmp'
                BASE_URL = "http://localhost:${STAGING_PORT}"
            }
            steps {
                sh 'pytest tests/test_site.py -v --junitxml=test-results.xml'
            }
            post {
                always {
                    junit 'test-results.xml'
                    archiveArtifacts artifacts: 'screenshot.png', allowEmptyArchive: true
                }
            }
        }
    }
    post {
        always {
            sh 'docker rm -f lab11-staging || true'
        }
    }
}
