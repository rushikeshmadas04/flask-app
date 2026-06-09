pipeline {

    agent any

    stages {

        stage('Build') {

            steps {

                echo 'Building Application'

                sh 'python3 --version'
            }
        }

        stage('Install Dependencies') {

            steps {

                sh 'pip3 install -r requirements.txt'
            }
        }

        stage('Test') {

            steps {

                echo 'Running Tests'

                sh 'pytest'
            }
        }
    }

    post {

        success {

            echo 'Build Successful'
        }

        failure {

            echo 'Build Failed'
        }

        always {

            echo 'Pipeline Finished'
        }
    }
}

