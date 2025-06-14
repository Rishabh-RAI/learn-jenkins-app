pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID='88115a96-3ec8-485a-aaa4-c0e00d402123'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        stage('Build') {
            agent{
                docker{
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
        stage('Run Test'){
            parallel{
                stage('E2E'){
                            agent{
                                docker{
                                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                    reuseNode true
                                
                                }
                            }
                            steps {
                                sh'''
                                    npm install serve
                                    node_modules/.bin/serve -s build & 
                                    sleep 10
                                    npx playwright test --reporter=html
                                '''
                            }
                            post {
                                always{
                                    junit 'jest-results/junit.xml'
                                }
                              }
                        }
                stage('Unit Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh'''
                            test -f build/index.html
                            npm test
                        '''
                    }
                post {
                    always{
                        junit 'jest-results/junit.xml'
                    }
                    }
                }
            }
        }
        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Site_Id = $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=/var/jenkins_home/workspace/learn-jenkins-app/build --prod
                '''
            }
        }
    }

}
