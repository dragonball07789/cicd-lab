
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
