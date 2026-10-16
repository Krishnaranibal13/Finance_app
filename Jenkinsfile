pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        PROJECT_DIR = '/home/ubuntu/Finance_app'
        COMPOSE_FILE = 'docker-compose.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Configuration') {
            steps {
                sh '''
                    set -eu
                    test -f "$COMPOSE_FILE"
                    test -f sql/Dockerfile
                    test -f backend/Dockerfile
                    test -f frontend/Dockerfile
                    docker --version
                    docker compose version
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -eu
                    docker compose -f "$COMPOSE_FILE" build
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    set -eu

                    if [ ! -d "$PROJECT_DIR" ]; then
                        echo "Project directory does not exist: $PROJECT_DIR"
                        exit 1
                    fi

                    cd "$PROJECT_DIR"

                    docker compose -f "$COMPOSE_FILE" up -d --build
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu
                    cd "$PROJECT_DIR"

                    docker compose -f "$COMPOSE_FILE" ps
                    sleep 10

                    docker compose -f "$COMPOSE_FILE" ps --services --filter status=running
                    docker compose -f "$COMPOSE_FILE" logs --tail=50
                '''
            }
        }
    }

    post {
        success {
            echo 'Finance_app deployment pipeline completed successfully.'
        }
        failure {
            echo 'Deployment failed. Check the stage logs in Jenkins.'
        }
        always {
            sh '''
                cd "$PROJECT_DIR" 2>/dev/null || exit 0
                docker compose -f "$COMPOSE_FILE" ps || true
            '''
        }
    }
}
