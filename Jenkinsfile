pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'a362ebc9-dc6a-4ec3-a871-75524f228f44'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                    npm run build 
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ''' 
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('e2e') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ''' 
                            npm install serve
                            npx serve -s build -l 3000 &
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
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh ''' 
                    npm install netlify-cli
                    npx netlify --version
                    npx netlify status
                    npx netlify deploy --dir=build
                '''
            }
        }

        stage ('Approval') {
            steps{
                input 'Ready to deploy ? '
            }
        }
        stage('Deploy Production') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh ''' 
                    npm install netlify-cli
                    npx netlify --version
                    npx netlify status
                    npx netlify deploy --dir=build --prod
                '''
            }
        }


        stage('Prod-e2e') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    environment {
                            CI_ENVIRONMENT_URL = 'https://preeminent-froyo-987b10.netlify.app/'
                                }
                    steps {
                        sh ''' 
                            
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Production HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }


        
    }

    post {
        always {
            sh 'ls -R'
        }
    }
}
