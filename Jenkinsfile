@Library(['JenkinsTesLib', 'JenkinsTesLib@master']) _

DeployToMaster(anyparam: "anyvalue")

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

        stage('push') {
            steps {
                script {
                    def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
                    def localImage = isMain ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    def remoteImage = isMain ? 'kzhanuzak/node:main-v1.0' : 'kzhanuzak/node:dev-v1.0'
                    
                    withCredentials([usernamePassword(credentialsId: 'jen-docker', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        bat "docker login -u %USER% -p %PASS%"
                        bat "docker tag ${localImage} ${remoteImage}"
                        bat "docker push ${remoteImage}"
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

        stage('deploy') {
            steps {
                def isMain = (env.BRANCH_NAME == 'main' || env.GIT_BRANCH == 'origin/main')
                def targetJob = isMain ? 'deploy_to_main' : 'deploy_to_dev'
                DeployToMaster(anyparam: targetJob)
            }
        }
    }
}