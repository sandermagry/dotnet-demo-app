pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                sh '''
                    docker stop todoapp todoappdb samplerunning || true
                    docker rm todoapp todoappdb samplerunning || true
                    docker network create todo-net || true
                '''
            }
        }

        stage('Start Database') {
            steps {
                sh '''
                    docker run -d \
                      --name todoappdb \
                      --network todo-net \
                      -e MARIADB_ROOT_PASSWORD=sekrit \
                      -e MARIADB_DATABASE=todo_db \
                      -e MARIADB_USER=todo_usr \
                      -e MARIADB_PASSWORD=letmeinplz \
                      mariadb:11

                    # Wacht even tot MariaDB gereed is om verbindingen te accepteren
                    sleep 10

                    # Laad schema.sql in via docker exec
                    docker exec -i todoappdb mariadb -u todo_usr -pletmeinplz todo_db < ./TodoApp/schema.sql
                '''
            }
        }

        stage('Build & Run App') {
            steps {
                sh '''
                    docker build -t todoapp:latest ./TodoApp

                    docker run -d \
                      --name todoapp \
                      --network todo-net \
                      -p 5051:8080 \
                      -e ConnectionStrings__TodoDb="Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz;" \
                      -e ASPNETCORE_ENVIRONMENT=Development \
                      todoapp:latest
                '''
            }
        }

        stage('Wait for App') {
            steps {
                sh 'sleep 10'
            }
        }

        stage('Test') {
            steps {
                sh 'curl -s -I http://172.16.0.10:5051/ | grep "HTTP"'
            }
        }
    }

    post {
        failure {
            sh '''
                docker logs todoappdb --tail=30 || true
                docker logs todoapp --tail=30 || true
            '''
        }
    }
}
