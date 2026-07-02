pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS'
    }
    
    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        stage('build') {
            steps {
                script {
                    // Update the asset based on the branch before running application build
                    if (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main') {
                        sh "echo '<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 100 100\"><circle cx=\"50\" cy=\"50\" r=\"40\" fill=\"blue\"/><text x=\"50\" y=\"55\" font-size=\"10\" text-anchor=\"middle\" fill=\"white\">MAIN</text></svg>' > logo.svg"
                    } else {
                        sh "echo '<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 100 100\"><rect x=\"10\" y=\"10\" width=\"80\" height=\"80\" fill=\"orange\"/><text x=\"50\" y=\"55\" font-size=\"10\" text-anchor=\"middle\" fill=\"white\">DEV</text></svg>' > logo.svg"
                    }
                }
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
                    def imageName = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    def hostPort = isMain ? '3000' : '3001'
                    def containerName = isMain ? 'node-app-main' : 'node-app-dev'

                    // Safe force remove to clean up previous containers and avoid deployment failures
                    sh "docker rm -f ${containerName} 2>/dev/null || true"
                    sh "docker run -d --name ${containerName} -p ${hostPort}:3000 ${imageName}"
                }
            }
        }
    }
}