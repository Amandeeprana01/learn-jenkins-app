pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '8686a009-5883-4cea-bb3d-243c44bcd1fa'
        NETLIFY_AUTH_TOKEN = credentials('netlify-tocken')
        CI_ENVIRONMENT_URL= 'https://celebrated-genie-34fe47.netlify.app/'
    }

    stages {

        stage('Docker') {
            step {
                sh 'docker build -t my-playwright .'
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
                npm install serve  # ✅ Local install of serve
                npm run build
                ls -la
                '''
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        test -f build/index.html
                        npm test
                        '''
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install
                        npm run build
                        npx serve -s build &
                        sleep 10
                        npx playwright test --reporter=html --output=playwright-report
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Local',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL= 'STAGING_URL_TO_BE'
            }

            steps {
                sh '''
                npm install netlify-cli node-jq
                npx netlify --version
                echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                npx netlify status
                npx netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(npx node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test --reporter=html --output=playwright-report
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL= 'https://celebrated-genie-34fe47.netlify.app'
            }

            steps {
                sh '''
                node --version
                npm install netlify-cli
                npx netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                npx netlify status
                npx netlify deploy --dir=build --prod
                npx playwright test --reporter=html --output=playwright-report
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Production E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}
