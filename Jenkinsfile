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
            sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
            '''
        }
    }

    stage('Test') {
        steps {
            echo 'Running Tests'
            sh '''
                . venv/bin/activate
                pytest
            '''
            sh 'pytest --html=report.html'
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
        archiveArtifacts artifacts: 'report.html'
        echo 'Pipeline Finished'
    }

}

}