pipeline {
    agent any
    environment {
        REACT_APP_VERSION="1.0.$BUILD_ID"
        AWS_DEFAULT_REGION='eu-west-1'
        AWS_ECS_CLUSTER="LearnJenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE="LearnJenkinsApp-Service-Prod"
        AWS_ECS_TASK_DEFINITION="LearnJenkinsApp-TaskDefinition-Prod"
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
        stage('Build Docker Image') {
            steps {
                sh 'docker image build -t myjenkinsapp .'
            }
        }
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli:2.13.2'
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-id', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    yum install jq -y
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TASK_DEFINITION:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }
    }
}