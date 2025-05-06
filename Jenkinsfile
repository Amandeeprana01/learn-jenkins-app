pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '8686a009-5883-4cea-bb3d-243c44bcd1fa'
        NETLIFY_AUTH_TOKEN = credentials('netlify-tocken')
        CI_ENVIRONMENT_URL = 'https://celebrated-genie-34fe47.netlify.app/'
        AWS_DEFAULT_REGION = "us-east-1"
    }

    stages {
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json
                        aws ecs update-service  --cluster LearnJenkinsApp-Cluster --service LearnJenkinsApp-TaskDefiniton-Prod-service-0gavllr0 --task-definition LearnJenkinsApp-TaskDefiniton-Prod:2
                    '''
                }
            }
        }

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
    }
}
