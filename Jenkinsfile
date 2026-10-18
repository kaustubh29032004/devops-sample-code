
pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://jenkins-dind:2375'
        DOCKER_CLI = '/var/jenkins_home/docker'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Creating Python environment and installing dependencies...'
                sh '''
                    $DOCKER_CLI run --rm \
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
                    $DOCKER_CLI run --rm \
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
                    cp "$WORKSPACE/app.py" "$WORKSPACE/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting application in Docker...'
                sh '''
                    $DOCKER_CLI run -d --rm \
                        --name python-flask-app \
                        -p 5001:5000 \
                        -v "$WORKSPACE/python-app-deploy:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pip install flask==2.1.2 && python app.py"
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh '''
                    $DOCKER_CLI run --rm \
                        --network host \
                        -v "$WORKSPACE:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pip install -r requirements.txt && python test_app.py"
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
        always {
            sh '''
                $DOCKER_CLI rm -f python-flask-app 2>/dev/null || true
            '''
        }
    }
}
