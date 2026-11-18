
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code retrieved from GitHub!'
            }
        }

        stage('Build') {
            steps {
                echo 'Building my first CI/CD website...'
                sh 'ls -l'
                sh 'cat index.html'
            }
        }

        stage('Test') {
            steps {
                echo 'Checking website files...'
                sh 'test -f index.html'
                echo 'Website file exists. Test passed!'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build -t cicd-website:v1 .
                '''

                echo 'Docker image built successfully!'
            }
        }
        
        stage('Docker Push') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        set -eu

                        echo "$DOCKERHUB_TOKEN" | docker login \
                            --username "$DOCKERHUB_USERNAME" \
                            --password-stdin

                        docker tag cicd-website:v1 \
                            "$DOCKERHUB_USERNAME/cicd-website:v1"

                        docker push \
                            "$DOCKERHUB_USERNAME/cicd-website:v1"
                        
                        docker logout
                    
                    '''
                }

                echo 'Docker image pushed successfully!'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying website using Ansible...'

                sh '''
                    ansible-playbook \
                    -i inventory.ini \
                    deploy.yml
                '''
            }
        }
        
        stage('Docker Deploy') {
            steps {
                echo 'Deploying Docker container to server2...'

                sh '''
                    ansible-playbook \
                    -i inventory.ini \
                    deploy-docker.yml
                '''

                echo 'Docker deployment completed!'
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployed website...'

                sh '''
                    set -eu

                    response=$(curl -fsS http://site4.lab.test)

                    echo "$response"

                    echo "$response" | grep -q '<h1>'

                    echo 'Website verification passed!'
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'FAILURE: CI/CD Pipeline failed. Check the Console Output!'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
