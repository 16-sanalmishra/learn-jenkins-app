pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'cbb94c54-35d0-4f68-b9bb-904fb9530ef7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        
    }

    stages {
        stage('Docker') {
            steps {
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
                echo "Small Change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
        
        stage('Run Tests'){
            parallel {
                    stage('Test') {
                        agent {
                            docker {
                                    image 'node:18-alpine'
                            reuseNode true
                            }
                        }
                        steps {
                            echo "Running Test Stage"
                            sh '''
                                test -f build/index.html
                                npm test
                            '''
                        }
                    }
                    stage('E2E') {
                        agent {
                            docker {
                                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                reuseNode true
                            }
                        }
                        steps {
                        
                            sh '''
                                npm install serve
                                node_modules/.bin/serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }
                            
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

            }

        }

        stage('Deploy Staging') { 
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = 'TO_BE_SET'
                    }
                    steps {
                        
                        sh '''
                            npm install netlify-cli node-jq
                            node_modules/.bin/netlify --version
                            echo "Deployiing to production. Site ID : $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                            CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                            npx playwright test --reporter=html
                            
                        '''
                    }
                            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input message: 'Do you wish to deploy?', ok: 'Yes I am sure'
        //         }       
        //     }
        // }
        
        stage('Deploy Prod') { 
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    environment {
                        CI_ENVIRONMENT_URL = 'https://storied-gaufre-698773.netlify.app'
                    }
                    steps {
                        
                        sh '''
                            node --version
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                            echo "Deployiing to production. Site ID : $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod
                            npx playwright test --reporter=html
                        '''
                    }
                            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlaywrightE2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

       
    }
    
}
