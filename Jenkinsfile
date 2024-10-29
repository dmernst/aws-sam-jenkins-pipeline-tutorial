pipeline {
    agent any

    stages {
        stage('Install sam-cli') {
            steps {
                // Set up a Python virtual environment and install the AWS SAM CLI
                sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'

                // Update npm to the latest version compatible with Node.js 12
                sh 'npm install -g npm@latest-12' 

                // Stash the virtual environment for later stages
                stash includes: '**/venv/**/*', name: 'venv'
            }
        }

        stage('Build') {
            steps {
                // Unstash the virtual environment
                unstash 'venv'

                // Verify npm version to ensure the update was successful
                sh 'npm -v'

                // Build the SAM application
                sh 'venv/bin/sam build'

                // Stash the built application
                stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
            }
        }

        stage('beta') {
            environment {
                STACK_NAME = 'sam-app-beta-stage'
                S3_BUCKET = 'sam-jenkins-demo-us-west-2-dmernst'
            }
            steps {
                // Set AWS credentials and region for deployment
                withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-west-2') {
                    // Unstash the virtual environment and build artifacts
                    unstash 'venv'
                    unstash 'aws-sam'

                    // Deploy to the beta environment
                    sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'

                    // Navigate to the directory and run npm commands for tests
                    dir ('hello-world') {
                        sh 'npm ci'
                        sh 'npm run integ-test'
                    }
                }
            }
        }

        stage('prod') {
            environment {
                STACK_NAME = 'sam-app-prod-stage'
                S3_BUCKET = 'sam-jenkins-demo-us-east-1-dmernst'
            }
            steps {
                // Set AWS credentials and region for deployment
                withAWS(credentials: 'sam-jenkins-demo-credentials', region: 'us-east-1') {
                    // Unstash the virtual environment and build artifacts
                    unstash 'venv'
                    unstash 'aws-sam'

                    // Deploy to the production environment
                    sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
                }
            }
        }
    }
}
