pipeline {
    agent {
        docker {
            image 'python:3.9-slim'
            args '-u root'
        }
    }

    stages {
        stage('Build') {
            steps {
                echo 'Installing dependencies...'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'python -m unittest discover -s .'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    mkdir -p "${WORKSPACE}/python-app-deploy"
                    cp app.py "${WORKSPACE}/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting application...'
                sh '''
                    nohup python "${WORKSPACE}/python-app-deploy/app.py" \
                        > "${WORKSPACE}/python-app-deploy/app.log" 2>&1 &
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh 'python test_app.py'
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