
```groovy
pipeline {
    agent any

    environment {
        PHP = '/usr/bin/php'
    }

    stages {
        stage('Preparation') {
            steps {
                // Checkout code from GitHub repository
                git 'https://github.com/manimohali/demo.git'
                
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Composer dependencies
                sh '''
                    export PATH=$PHP/bin:$PATH
                    curl -sS https://getcomposer.org/installer | php
                    php composer.phar install
                '''
            }
        }

        stage('Code Sniff') {
            steps {
                // Run PHPCS
                sh 'vendor/bin/phpcs --standard=PSR2 src/'
            }
        }

        stage('Unit Tests') {
            steps {
                // Run PHPUnit
                sh 'vendor/bin/phpunit tests'
            }
        }

    }
    
     post {
        always {
            // Archive test results and artifacts
            junit 'tests/_output/*.xml'
            archiveArtifacts artifacts: 'src/**/*', allowEmptyArchive: true
        }
    }

}

```