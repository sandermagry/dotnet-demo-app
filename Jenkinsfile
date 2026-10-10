pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                sh '''
                    # Stop en verwijder eerdere containers als ze bestaan
                    docker stop todoapp todoappdb || true
                    docker rm todoapp todoappdb || true
                    # Zorg voor een gedeeld netwerk zodat app en db elkaar vinden
                    docker network create todo-net || true
                '''
            }
        }

        stage('Start Database') {
            steps {
                sh '''
                    # Start MariaDB met dezelfde variabelen als in docker-compose.yml
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
                    # Bouw de .NET image aan de hand van de Dockerfile
                    docker build -t todoapp:latest ./TodoApp

                    # Start de todoapp op poort 5050 verbonden met hetzelfde netwerk
                    docker run -d \
                      --name todoapp \
                      --network todo-net \
                      -p 5050:8080 \
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
                // Voer de acceptance test uit
                sh 'curl -s -I http://172.16.0.10:5050/ | grep "HTTP"'
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
