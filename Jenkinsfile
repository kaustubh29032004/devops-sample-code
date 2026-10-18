
pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://jenkins-dind:2375'
        DOCKER = '/var/jenkins_home/docker'
        APP_DIR = '.'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    test -f app.py
                    test -f requirements.txt
                    test -f test_app.py

                    "$DOCKER" run --rm \
                        -v "$WORKSPACE:/app" \
                        -w /app \
                        python:3.9-slim \
                        pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    "$DOCKER" run --rm \
                        -v "$WORKSPACE:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pip install -r requirements.txt && python -m unittest discover -s ."
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    mkdir -p "$WORKSPACE/python-app-deploy"
                    cp app.py requirements.txt "$WORKSPACE/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
                sh '''
                    "$DOCKER" rm -f python-flask-app || true

                    "$DOCKER" run -d \
                        --name python-flask-app \
                        -p 5001:5000 \
                        -v "$WORKSPACE/python-app-deploy:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pip install -r requirements.txt && python app.py"
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Running final application tests...'
                sh '''
                    "$DOCKER" run --rm \
                        -v "$WORKSPACE:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pip install -r requirements.txt && python -m unittest discover -s ."
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
