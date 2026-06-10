pipeline {

    // Use a Docker-enabled agent so you don't need Docker installed in Jenkins container
    agent {
        docker {
            image 'python:3.11-slim'
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        VENV_PATH = "${WORKSPACE}/venv"
    }

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
                    python3 -m venv $VENV_PATH
                    . $VENV_PATH/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . $VENV_PATH/bin/activate
                    pytest --html=report.html
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t flask-demo:v1 .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker tag flask-demo:v1 madasrushi0804/flask-demo:v1
                    docker push madasrushi0804/flask-demo:v1
                '''
            }
        }

        stage('Kubernetes Connectivity Test') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl config current-context
                        kubectl get nodes
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')
                ]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl apply -f k8s/
                        kubectl rollout status deployment/flask-demo
                        kubectl get pods
                        kubectl get svc
                    '''
                }
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
            archiveArtifacts artifacts: 'report.html', allowEmptyArchive: true
            echo 'Pipeline Finished'
        }
    }

}