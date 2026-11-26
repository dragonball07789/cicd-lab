
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
        
        stage('ECR Push') {
            steps {
                echo 'Pushing Docker image to AWS ECR...'

                withCredentials([usernamePassword(
                    credentialsId: 'aws-ecr-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set +x
                        set -eu

                        export AWS_DEFAULT_REGION=ap-southeast-2

                        ECR_REGISTRY=996009836361.dkr.ecr.ap-southeast-2.amazonaws.com
                        ECR_REPOSITORY=my-nginx-website
                        ECR_IMAGE="$ECR_REGISTRY/$ECR_REPOSITORY"

                        echo "Logging in to AWS ECR..."

                        aws ecr get-login-password --region "$AWS_DEFAULT_REGION" | \
                            docker login --username AWS --password-stdin "$ECR_REGISTRY"

                        echo "Tagging Docker image..."

                        docker tag cicd-website:v1 "$ECR_IMAGE:$BUILD_NUMBER"

                        echo "Pushing image to ECR..."

                        docker push "$ECR_IMAGE:$BUILD_NUMBER"

                        docker logout "$ECR_REGISTRY"
                    '''
                }

                echo 'Image successfully pushed to AWS ECR!'
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
                echo 'Verifying both website deployments...'

                sh '''
                    set -eu

                    echo "Checking server1..."
                    response1=$(curl -fsS http://site4.lab.test)
                    echo "$response1" | grep -q '<h1>'
                    echo "server1 website verification passed!"

                    echo "Checking server2 Docker container..."
                    response2=$(curl -fsS http://192.168.58.11:8081)
                    echo "$response2" | grep -q '<h1>'
                    echo "server2 Docker website verification passed!"
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
