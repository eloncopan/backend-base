pipeline {
    agent any
    environment {
        USERNAME = 'cmd'
    }
    options {
        disableConcurrentBuilds()
    }
    stages {
        stage('Build and test') {
            agent {
                docker {
                    image 'node:20.11.1-alpine3.19' 
                    reuseNode true
                }
            }
            stages {
               stage('Instalar dependencias') {
                   steps {
                       sh 'npm install'
                   }
               } 
                stage('ejecucion de test') {
                   steps {
                       sh 'npm run test'
                   }
               } 
                stage('ejecucion de build') {
                   steps {
                       sh 'npm run build'
                   }
               } 
            }
        }
        stage('delivery'){
            steps {
                script {
                    docker.withRegistry('http://localhost:8082', 'nexus-key') {
                        sh 'docker build -t backend-base:latest .'
                        sh "docker tag backend-base:latest localhost:8082/backend-base:latest"
                        sh "docker tag backend-base:latest localhost:8082/backend-base:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                        sh 'docker push localhost:8082/backend-base:latest'
                        sh "docker push localhost:8082/backend-base:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('deploy'){
            steps {
                script {
                    
                    if (env.BRANCH_NAME == 'main') {
                        ambiente = 'prd'
                    } else {
                        ambiente = 'dev'
                    }

                    docker.withRegistry('http://localhost:8082', 'nexus-key') {
                        withCredentials([file(credentialsId: "${ambiente}-env", variable: 'ENV_FILE')]) {
                            sh "docker compose pull"
                            sh "docker compose --env-file ENV_FILE -d up --force-recreate --build  "
                        }
                    }
                }
            }
        }
    }
}