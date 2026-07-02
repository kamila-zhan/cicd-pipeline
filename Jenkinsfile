pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }
    
    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        stage('build') {
            steps {
                sh 'npm install'
            }
        }

        stage('test') {
            steps {
                sh 'npm test'
            }
        }

        stage('build docker image') {
            steps {
                script {
                    def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
                    def imageName = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('deploy') {
            steps {
                script {
                    def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
                    def name = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    def port = isMain ? '3000' : '3001'

                    sh "docker rm -f ${containerName} 2>/dev/null || true"
                    sh "docker run -d --expose ${port} -p ${port}:3000 ${name}"
                }
            }
        }
    }
}