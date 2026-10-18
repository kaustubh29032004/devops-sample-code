
pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://jenkins-dind:2375'
        DOCKER = '/var/jenkins_home/docker'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Locating Flask application and installing dependencies...'
                sh '''
                    set -eu

                    if [ -f "$WORKSPACE/app.py" ] &&
                       [ -f "$WORKSPACE/requirements.txt" ] &&
                       [ -f "$WORKSPACE/test_app.py" ]; then
                        APP_PATH="$WORKSPACE"
                    elif [ -f "$WORKSPACE/python-flask-app/app.py" ] &&
                         [ -f "$WORKSPACE/python-flask-app/requirements.txt" ] &&
                         [ -f "$WORKSPACE/python-flask-app/test_app.py" ]; then
                        APP_PATH="$WORKSPACE/python-flask-app"
                    else
                        echo "ERROR: Required Flask files not found."
                        find "$WORKSPACE" -maxdepth 4 -type f \
                            ! -path '*/.git/*' -print
                        exit 1
                    fi

                    echo "Application directory: $APP_PATH"
                    echo "$APP_PATH" > "$WORKSPACE/.flask_app_path"

                    "$DOCKER" run --rm \
                        -v "$APP_PATH:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "pwd && ls -la && python -m pip install -r requirements.txt"
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    set -eu
                    APP_PATH="$(cat "$WORKSPACE/.flask_app_path")"

                    "$DOCKER" run --rm \
                        -v "$APP_PATH:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "python -m pip install -r requirements.txt && python -m unittest discover -s ."
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    set -eu
                    APP_PATH="$(cat "$WORKSPACE/.flask_app_path")"
                    DEPLOY_PATH="$WORKSPACE/python-app-deploy"

                    mkdir -p "$DEPLOY_PATH"
                    cp "$APP_PATH/app.py" "$DEPLOY_PATH/"
                    cp "$APP_PATH/requirements.txt" "$DEPLOY_PATH/"
                    cp "$APP_PATH/test_app.py" "$DEPLOY_PATH/"

                    echo "Application copied to $DEPLOY_PATH"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
                sh '''
                    set -eu
                    "$DOCKER" rm -f python-flask-app || true

                    "$DOCKER" run -d \
                        --name python-flask-app \
                        -p 5001:5000 \
                        -v "$WORKSPACE/python-app-deploy:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "python -m pip install -r requirements.txt && python app.py"
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing the deployed application...'
                sh '''
                    set -eu

                    "$DOCKER" run --rm \
                        -v "$WORKSPACE/python-app-deploy:/app" \
                        -w /app \
                        python:3.9-slim \
                        sh -c "python -m pip install -r requirements.txt && python -m unittest discover -s ."
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
            echo 'Pipeline execution finished.'
        }
    }
}
