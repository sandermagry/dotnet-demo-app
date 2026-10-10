pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                sh '''
                    # Stop en verwijder oude containers
                    docker stop todoapp todoappdb samplerunning || true
                    docker rm todoapp todoappdb samplerunning || true
                    # Zorg voor een gedeeld netwerk
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
                      -v mariadb-data:/var/lib/mysql \
                      -v "$PWD/TodoApp/schema.sql":/docker-entrypoint-initdb.d/schema.sql:ro \
                      mariadb:11
                '''
            }
        }

        stage('Build & Run App') {
            steps {
                sh '''
                    docker build -t todoapp:latest ./TodoApp

                    # Gebruik poort 5051 om poortconflicten te vermijden
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
                sh 'sleep 15'
            }
        }

        stage('Test') {
            steps {
                // Test op poort 5051
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
