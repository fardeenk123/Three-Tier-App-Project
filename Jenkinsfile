pipeline {
    agent any
    environment {
        DockerhubUsername = 'fardeenk1234'
        FrontendImage = "${DockerhubUsername}/tasktracker-frontend"
        BackendImage  = "${DockerhubUsername}/tasktracker-backend"
        ImageTag      = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Repo') {
            steps {
                echo 'Cloning Github Repo ...'
                git branch: 'main',
                    credentialsId: 'GitHub-Creds',
                    url: 'https://github.com/fardeenk123/EmployeeTaskTracker.git'
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh "docker build --no-cache -t ${FrontendImage}:${ImageTag} ."
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh "docker build --no-cache -t ${BackendImage}:${ImageTag} ."
                }
            }
        }

        stage('Login To DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DockerhubUsername',
                    passwordVariable: 'DockerhubPassword'
                )]) {
                    sh "echo ${DockerhubPassword} | docker login -u ${DockerhubUsername} --password-stdin"
                }
            }
        }

        stage('Push To DockerHub') {
            steps {
                sh "docker push ${FrontendImage}:${ImageTag}"
                sh "docker push ${BackendImage}:${ImageTag}"
            }
        }

        stage('Run Containers') {
            steps {
                sh 'docker network create tasktracker-net || true'

                sh 'docker stop tasktracker-mysql    || true'
                sh 'docker stop tasktracker-backend  || true'
                sh 'docker stop tasktracker-frontend || true'
                sh 'docker rm   tasktracker-mysql    || true'
                sh 'docker rm   tasktracker-backend  || true'
                sh 'docker rm   tasktracker-frontend || true'
                
                withCredentials([
                    string(credentialsId: 'mysql-root-password', variable: 'MYSQL_ROOT_PASSWORD'),
                    string(credentialsId: 'db-user', variable: 'DB_USER'),
                    string(credentialsId: 'db-password', variable: 'DB_PASS')
                    ]){
                sh '''
                    docker run -d \
                        --name tasktracker-mysql \
                        --network tasktracker-net \
                        -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
                        -e MYSQL_DATABASE=task_tracker_db \
                        -e MYSQL_USER=${DB_USER} \
                        -e MYSQL_PASSWORD=${DB_PASS} \
                        -v mysql_data:/var/lib/mysql \
                        mysql:8.0
                '''

                sh 'sleep 30'

                sh """
                    docker run -d \
                        --name tasktracker-backend \
                        --network tasktracker-net \
                        -p 8080:8080 \
                        ${BackendImage}:${ImageTag}
                """
            }

                sh 'sleep 20'

                sh """
                    docker run -d \
                        --name tasktracker-frontend \
                        --network tasktracker-net \
                        -p 3000:3000 \
                        ${FrontendImage}:${ImageTag}
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful! '
        }
        failure {
            sh 'docker logs tasktracker-backend  || true'
            sh 'docker logs tasktracker-frontend || true'
            sh 'docker logs tasktracker-mysql    || true'
        }
    }
}
