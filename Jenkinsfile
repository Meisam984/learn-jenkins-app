pipeline {
    agent any
    environment {
        REACT_APP_VERSION="1.0.$BUILD_ID"
        APP_NAME='learnjenkinsapp'
        AWS_DEFAULT_REGION='eu-west-1'
        AWS_ECS_CLUSTER="LearnJenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE="LearnJenkinsApp-Service-Prod"
        AWS_ECS_TASK_DEFINITION="LearnJenkinsApp-TaskDefinition-Prod"
        AWS_ECR_REPO='905418032282.dkr.ecr.eu-west-1.amazonaws.com'
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
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-id', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        docker image build -t $AWS_ECR_REPO/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_ECR_REPO
                        docker image push $AWS_ECR_REPO/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-id', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version 
                    sed -i "s/#APP_VERSION#/$REACTA_APP_VERSION/g" aws/task-definition-prod.json
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