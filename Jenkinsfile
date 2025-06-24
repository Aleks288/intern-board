pipeline {
    agent { label 'my-agent' }

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION = 'eu-central-1'
        S3_BUCKET = 'symfony-builds-aleks-2025'

        GIT_COMMIT_SAFE = "${env.GIT_COMMIT ?: 'unknown-commit'}"
        GIT_AUTHOR = "${env.GIT_AUTHOR ?: 'unknown-author'}"
        DATE_STRING = new Date().format('yyyyMMdd-HHmmss', TimeZone.getTimeZone('UTC'))
        ARCHIVE_NAME = "symfony-app-${DATE_STRING}.tar.gz"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Lint') {
            steps {
                echo "Running PHP lint"
                sh 'find src -type f -name "*.php" -exec php -l {} \\;'
            }
        }

        stage('Test') {
            when {
                changeRequest()   // For every PR
            }
            steps {
                echo "Running tests"
                sh 'composer install --no-interaction'
                sh './vendor/bin/phpunit'
            }
        }

        stage('Package & Push to S3') {
            when {
                allOf {
                    not { changeRequest() } // It is not PR and changes are pushed to the master (PR merged to master)
                    branch 'master'
                }
            }
            steps {
                script {
                    def path = "symfony-builds/${DATE_STRING}-${GIT_AUTHOR}"
                    echo "Archiving and uploading to S3 path: ${path}"
                    sh """
                    mkdir -p /tmp/build-copy
                    rsync -a --exclude='.git' ./ /tmp/build-copy/
                        tar --exclude=".git" -czf ${ARCHIVE_NAME} /tmp/build-copy/
                        rm -rf /tmp/build-copy
                        aws s3 cp ${ARCHIVE_NAME} s3://${S3_BUCKET}/${path}/
                    """
                }
            }
        }
    }
}