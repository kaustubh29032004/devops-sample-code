
pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://jenkins-dind:2375'
        DOCKER_CLI = '/var/jenkins_home/docker'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Locating application and installing dependencies...'
                sh '''
                    if [ -f "$WORKSPACE/requirements.txt" ]; then
                        APP_PATH="$WORKSPACE"
                    elif [ -f "$WORKSPACE/python-flask-app/requirements.txt" ]; then
                        APP_PATH="$WORKSPACE/python-flask-app"
                    else
                        echo "ERROR: requirements.txt not found."
                        echo "Workspace contents:"
                        find "$WORKSPACE" -maxdepth 4 -type f
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
