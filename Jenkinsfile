pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
                sh '''
                    set -eu
                    python3 -m venv venv
                    . venv/bin/activate

                    REQ_PATH="$WORKSPACE/requirements.txt"
                    if [ ! -f "$REQ_PATH" ] && [ -f "$WORKSPACE/python-flask-app/requirements.txt" ]; then
                        REQ_PATH="$WORKSPACE/python-flask-app/requirements.txt"
                    fi

                    pip install --upgrade pip
                    pip install -r "$REQ_PATH"
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    set -eu
                    . venv/bin/activate

                    TEST_PATH="$WORKSPACE"
                    if [ ! -f "$WORKSPACE/test_app.py" ] && [ -f "$WORKSPACE/python-flask-app/test_app.py" ]; then
                        TEST_PATH="$WORKSPACE/python-flask-app"
                    fi

                    python3 -m unittest discover -s "$TEST_PATH"
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    set -eu
                    APP_SRC="$WORKSPACE"
                    if [ ! -f "$WORKSPACE/app.py" ] && [ -f "$WORKSPACE/python-flask-app/app.py" ]; then
                        APP_SRC="$WORKSPACE/python-flask-app"
                    fi

                    DEPLOY_PATH="$WORKSPACE/python-app-deploy"
                    mkdir -p "$DEPLOY_PATH"
                    cp "$APP_SRC/app.py" "$DEPLOY_PATH/"
                    cp "$APP_SRC/requirements.txt" "$DEPLOY_PATH/"
                    if [ -f "$APP_SRC/test_app.py" ]; then
                        cp "$APP_SRC/test_app.py" "$DEPLOY_PATH/"
                    fi

                    echo "Application copied to $DEPLOY_PATH"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
                sh '''
                    set -eu
                    DEPLOY_PATH="$WORKSPACE/python-app-deploy"
                    pkill -f "python3.*python-app-deploy/app.py" || true
                    export JENKINS_NODE_COOKIE=dontKillMe
                    nohup "$WORKSPACE/venv/bin/python3" "$DEPLOY_PATH/app.py" > "$WORKSPACE/app.log" 2>&1 &
                    sleep 3
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing deployed application...'
                sh '''
                    set -eu
                    DEPLOY_PATH="$WORKSPACE/python-app-deploy"
                    if ! curl -sS http://127.0.0.1:5000/ > /dev/null 2>&1; then
                        export JENKINS_NODE_COOKIE=dontKillMe
                        nohup "$WORKSPACE/venv/bin/python3" "$DEPLOY_PATH/app.py" > "$WORKSPACE/app.log" 2>&1 &
                        sleep 3
                    fi
                    echo "Response from deployed Flask application:"
                    curl -sS -i http://127.0.0.1:5000/
                '''
            }
        }
    }

    post {
        always {
            sh 'pkill -f "python3.*python-app-deploy/app.py" || true'
            echo 'Pipeline execution finished.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
