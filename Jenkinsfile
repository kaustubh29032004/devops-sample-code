
pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://jenkins-dind:2375'
        DOCKER = '/var/jenkins_home/docker'
        APP_DIR = 'python-flask-app'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    test -f "$APP_DIR/requirements.txt"
                    test -f "$APP_DIR/app.py"
                    test -f "$APP_DIR/test_app.py"

                    "$DOCKER" run --rm \
                        -v "$WORKSPACE/$APP_DIR:/app" \
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
                        -v "$WORKSPACE/$APP_DIR:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pip install -r requirements.txt && python -m unittest discover -s ."
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Copying application to deployment directory...'
                sh '''
                    mkdir -p "$WORKSPACE/python-app-deploy"
                    cp "$APP_DIR/app.py" "$WORKSPACE/python-app-deploy/"
                    cp "$APP_DIR/requirements.txt" "$WORKSPACE/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application in Docker...'
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
                echo 'Checking the running Flask application...'
                sh '''
                    "$DOCKER" run --rm \
                        --network host \
                        -v "$WORKSPACE/$APP_DIR:/app" \
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
