
pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://jenkins-dind:2375'
        DOCKER_CLI = '/var/jenkins_home/docker'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Finding application files and installing dependencies...'
                sh '''
                    echo "=== Repository files ==="
                    find "$WORKSPACE" -maxdepth 5 -type f

                    REQ_FILE=$(find "$WORKSPACE" -maxdepth 5 \
                        -type f -name requirements.txt \
                        ! -path '*/.git/*' | head -n 1)

                    if [ -z "$REQ_FILE" ]; then
                        echo "ERROR: requirements.txt is missing from the Git checkout."
                        exit 1
                    fi

                    APP_PATH=$(dirname "$REQ_FILE")

                    if [ ! -f "$APP_PATH/app.py" ] || [ ! -f "$APP_PATH/test_app.py" ]; then
                        echo "ERROR: app.py or test_app.py is missing from $APP_PATH"
                        exit 1
                    fi

                    echo "$APP_PATH" > "$WORKSPACE/.app_path"

                    $DOCKER_CLI run --rm \
                        -v "$APP_PATH:/app" \
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
                    APP_PATH=$(cat "$WORKSPACE/.app_path")

                    $DOCKER_CLI run --rm \
                        -v "$APP_PATH:/app" \
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
                    APP_PATH=$(cat "$WORKSPACE/.app_path")
                    mkdir -p "$WORKSPACE/python-app-deploy"
                    cp "$APP_PATH/app.py" "$WORKSPACE/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
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
                echo 'Testing Flask application...'
                sh '''
                    APP_PATH=$(cat "$WORKSPACE/.app_path")

                    $DOCKER_CLI run --rm \
                        --network host \
                        -v "$APP_PATH:/app" \
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
