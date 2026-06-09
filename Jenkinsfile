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
                sh '''
                    . venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t flask-demo:v1 .
                '''
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
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker tag flask-demo:v1 asrdigi/flask-demo:v1
                    docker push asrdigi/flask-demo:v1
                '''
            }
        }

        stage('Kubernetes Connectivity Test') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG_FILE'
                    )
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
                    file(
                        credentialsId: 'kubeconfig',
                        variable: 'KUBECONFIG_FILE'
                    )
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
            echo 'Pipeline Finished'
        }
    }
}