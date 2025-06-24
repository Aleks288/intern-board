pipeline {
    agent { label 'my-agent' }

    stages {
        stage('Lint') {
            steps {
                echo "Running PHP lint"
                sh 'find src -type f -name "*.php" -exec php -l {} \\;'
            }
        }

        stage('Install dependencies') {
            when {
                changeRequest()
            }
            steps {
                sh 'composer install --no-interaction'
            }
        }

        stage('Test') {
            when {
                changeRequest()
            }
            steps {
                sh './vendor/bin/phpunit'
            }
        }
    }
}