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
                bat 'npm install'
            }
        }

        stage('test') {
            steps {
                bat 'npm test'
            }
        }

        stage('build docker image') {
            steps {
                script {
                    def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
                    def imageName = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    bat "docker build -t ${imageName} ."
                }
            }
        }

        stage('deploy') {
            steps {
                script {
                    def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
                    def name = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    def port = isMain ? '3000' : '3001'
                    def containerName = isMain ? 'node-app-main' : 'node-app-dev'

                    bat "docker rm -f ${containerName}"
                    bat "docker run -d --name ${containerName} --expose ${port} -p ${port}:3000 ${name}"
                }
            }
        }
    }
}