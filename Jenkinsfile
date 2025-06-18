pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '8bd4f567-8005-4acc-b630-4c49b1f471bf'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        
        stage ('Build') {
            agent {
                docker {
                    image 'node:20.12.2-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                rm -rf build
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }
        }
        
        stage ('TEST') {
            parallel {
                stage ('UNIT Test') {
                    agent {
                        docker {
                            image 'node:20.12.2-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }

                }
                stage ('e2e') {
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
                            sleep 20
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright - HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }               
            }
        }

        stage ('Deploy') {
            agent {
                docker {
                    image 'node:20.12.2-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                   npm install netlify-cli@latest
                   node_modules/.bin/netlify --version
                   node_modules/.bin/netlify status
                   ./node_modules/.bin/netlify deploy --dir=build --prod
                   echo "Deploying to production site id: $NETLIFY_SITE_ID"
                '''
            }
        }

    }

}