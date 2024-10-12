pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        
    }

    stages {

        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }

        stage('Deploy to AWS') {
                    agent {
                        docker {
                            image 'amazon/aws-cli'
                            reuseNode true
                            args "--entrypoint=''"
                            
                        }
                        
                    }
                    environment {
                        AWS_DEFAULT_REGION = 'us-east-1'
                    }
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                        
                        sh '''
                            aws --version
                            aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json

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
                echo "Small Change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
    }
    
    
}
