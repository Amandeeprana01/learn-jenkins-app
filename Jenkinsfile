pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '8686a009-5883-4cea-bb3d-243c44bcd1fa'
        NETLIFY_AUTH_TOKEN = credentials('netlify-tocken')
        CI_ENVIRONMENT_URL = 'https://celebrated-genie-34fe47.netlify.app/'
        AWS_DEFAULT_REGION = "us-east-1"
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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm install serve  # Local install of serve
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    reuseNode true
                    args '-v /var/run/docker.sock:/var/run/docker.sock '
                }
            }
            steps {
                sh '''
                docker --version 
                docker build  -t myjekinsapp .
                '''
            }
        }

        stage('Deploy to AWS') {
            
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service  --cluster LearnJenkinsApp-Cluster --service LearnJenkinsApp-TaskDefiniton-Prod-service-0gavllr0 --task-definition LearnJenkinsApp-TaskDefiniton-Prod:$LATEST_TD_REVISION
                       

                    '''
                }
            }
        }
    }
}
