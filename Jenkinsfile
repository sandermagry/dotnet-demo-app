pipeline {
    agent any

    stages {
        stage('Preparation') {
            steps {
                // Sluit eventuele eerdere containers netjes af
                sh 'docker compose down || true'
            }
        }

        stage('Build & Run') {
            steps {
                // Start de MariaDB container en bouw/start de todoapp container
                sh 'docker compose up --build -d'
            }
        }

        stage('Wait for App') {
            steps {
                // Wacht even tot de .NET app en database volledig gestart zijn
                sh 'sleep 10'
            }
        }

        stage('Test') {
            steps {
                // Controleer of de applicatie reageert op poort 5050
                sh 'curl -s -I http://172.16.0.10:5050/ | grep "HTTP"'
            }
        }
    }

    post {
        failure {
            sh 'docker compose logs --tail=50'
        }
    }
}
