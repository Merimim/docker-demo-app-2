pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'docker-demo-app'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('deploy compose file'){
            steps {
                withCredentials ([
                                 string(credentialsId:'mongo-db-username',variable:'MONGO_DB_USERNAME'),
                                 string(credentialsId:'mongo-db-password',variable:'MONGO_DB_PASSWORD')
                                 ]) {
                                     sh '''
                                        export DOCKER_TAG=${DOCKER_TAG}
                                        export MONGO_DB_USERNAME=${MONGO_ADMIN}
                                        export MONGO_DB_PWD=${MONGO_PASSWORD}
                                        docker compose -f compose.yaml up -d
                                        '''
                }

            }
        }
        stage('image cleanup') {
            steps {
                sh '''
                   docker images ${DOCKER_IMAGE} --format "{{.Repository}}:{{.Tag}}" | \
                   grep -v latest | \
                   xargs -r docker rmi -f
                   '''}
        }
    }
}