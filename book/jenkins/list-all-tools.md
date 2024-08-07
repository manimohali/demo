```groovy
pipeline {
    agent any

    tools {
        customTool 'myphp' // Using the custom tool named 'myphp'
    }

    environment {
        COMPOSER_HOME = "${env.WORKSPACE}/.composer"
        PATH = "$PATH:$myphp/bin" // Adding the myphp executable path to the PATH environment variable
    }

    stages {
        stage('Preparation') {
            steps {
                // Checkout code from GitHub repository
                git 'https://github.com/yourUsername/demo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Composer dependencies
                sh '''
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
                sh 'vendor/bin/phpunit --coverage-text'
            }
        }

        stage('Deploy') {
            steps {
                // Deploy to remote server
                withCredentials([sshUserPrivateKey(credentialsId: 'your-ssh-credential-id', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ssh -i $SSH_KEY user@remote-server "mkdir -p /path/to/deploy"
                        scp -i $SSH_KEY -r * user@remote-server:/path/to/deploy
                        ssh -i $SSH_KEY user@remote-server 'cd /path/to/deploy && echo "Deployed Successfully!"'
                    '''
                }
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