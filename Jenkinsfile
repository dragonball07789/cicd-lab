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
    }
}
