pipeline {
    agent { label 'my-agent' }

    stages {
        stage('Lint') {
            steps {
                sh 'find src -type f -name "*.php" -exec php -l {} \\;'
            }
        }
        stage('Test') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }
    }
}