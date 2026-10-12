
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        APP_SERVER = '172.31.29.35'
        DEPLOY_USER = 'ubuntu'
        DEPLOY_DIR = '/var/www/html'
        SSH_KEY = '/var/lib/jenkins/.ssh/app_deploy_key'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/syedfaru-555/AtmWebSite.git'
            }
        }

        stage('Validate Website') {
            steps {
                sh '''
                    set -eu

                    if [ ! -f index.html ]; then
                        echo "ERROR: index.html not found"
                        exit 1
                    fi

                    echo "Website files:"
                    find . -maxdepth 2 -type f \
                        ! -path './.git/*' -print
                '''
            }
        }

        stage('Verify SSH Connection') {
            steps {
                sh '''
                    set -eu

                    test -r "$SSH_KEY"

                    ssh -i "$SSH_KEY" \
                        -o BatchMode=yes \
                        -o IdentitiesOnly=yes \
                        "$DEPLOY_USER@$APP_SERVER" \
                        "hostname && test -d '$DEPLOY_DIR'"
                '''
            }
        }

        stage('Deploy to Nginx') {
            steps {
                sh '''
                    set -eu

                    rsync -avz \
                        --exclude='.git/' \
                        --exclude='Jenkinsfile' \
                        -e "ssh -i $SSH_KEY -o BatchMode=yes -o IdentitiesOnly=yes" \
                        ./ \
                        "$DEPLOY_USER@$APP_SERVER:$DEPLOY_DIR/"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    ssh -i "$SSH_KEY" \
                        -o BatchMode=yes \
                        -o IdentitiesOnly=yes \
                        "$DEPLOY_USER@$APP_SERVER" \
                        "test -s '$DEPLOY_DIR/index.html'"

                    echo "Deployment completed successfully"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Website deployed to App Server.'
        }
        failure {
            echo 'FAILED: Check the Jenkins Console Output.'
        }
    }
}
