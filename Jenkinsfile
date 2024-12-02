pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID='306e4120-3782-4e42-9ef9-e1282b5500b5'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
        REACT_APP_VERSION="1.0.$BUILD_ID"
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small Change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.13.2'
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET='sam-learn-jenkins-app'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-id', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    aws s3 ls
                    aws s3 sync build s3://sam-learn-jenkins-app
                    '''
                }
            }
        }
    }
}