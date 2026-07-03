@Library('JenkinsTestLib@main') _

pipeline {
    agent any
    
    tools {
        nodejs 'node'
    }

    environment {
        IS_MAIN = "${(env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')}"
        LOCAL_IMAGE = "${IS_MAIN ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        REMOTE_IMAGE = "${IS_MAIN ? 'kzhanuzak/node:main-v1.0' : 'kzhanuzak/node:dev-v1.0'}"
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
                    bat "docker build -t ${env.LOCAL_IMAGE} ."
                }
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                script {
                    def vulnerabilities = bat(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${env.LOCAL_IMAGE}", returnStdout: true).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
            }
        }

        stage('push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jen-docker', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        bat "docker login -u %USER% -p %PASS%"
                        bat "docker tag ${env.LOCAL_IMAGE} ${env.REMOTE_IMAGE}"
                        bat "docker push ${env.REMOTE_IMAGE}"
                    }
                }
            }
        }

        // stage('deploy') {
        //     steps {
        //         script {
        //             def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
        //             def name = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
        //             def port = isMain ? '3000' : '3001'
        //             def containerName = isMain ? 'node-app-main' : 'node-app-dev'

        //             bat "docker rm -f ${containerName}"
        //             bat "docker run -d --name ${containerName} --expose ${port} -p ${port}:3000 ${name}"
        //         }
        //     }
        // }

        // stage('deploy') {
        //     steps {
        //         script {
        //             def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
        //             def targetJob = isMain ? 'deploy_to_main' : 'deploy_to_dev'
        //             build job: targetJob, wait: false
        //         }
        //     }
        // }

        stage('Deploy') {
            steps {
                script {
                    Deploy() 
                }
            }
        } 
    }
}