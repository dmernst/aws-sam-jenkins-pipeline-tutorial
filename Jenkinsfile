pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = "${params.AWS_account_number}.dkr.ecr.${AWS_REGION}.amazonaws.com/loadtest-dltecsdltecr2419f66f-udaavaeude24"
        ECS_CLUSTER = 'jenkins-demo'
        ECS_SERVICE = 'jenkins-demo-service'
    }
    stages {
        stage('Build') {
            steps {
                echo "Building a minimal Docker image with scratch as the base..."
                echo "Creating a minimal C program file"
                
                sh '''
                  cat > hello.c << EOF
                  #include <stdio.h>
                  int main() {
                      printf("Hello, World!\\n");
                      return 0;
                  }
EOF
                '''
                
                sh 'gcc -o hello hello.c'
                
                echo "Creating Dockerfile for minimal image"
                sh '''
                  cat > Dockerfile << EOF
                  FROM scratch
                  COPY hello /
                  CMD ["/hello"]
EOF
                '''
            }
        }
        stage('Dockerize') {
            steps {
                echo "Creating Docker image from scratch"
                sh 'docker build -t my-dummy-image .'
            }
        }
        stage('Push') {
            steps {
                echo "Pushing Docker image to repository"
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS-creds']]) {
                    sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
                }
                sh 'docker tag my-dummy-image $ECR_REPO:latest'
                sh 'docker push $ECR_REPO:latest'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to ECS service"
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS-creds']]) {
                    sh '''
                    aws ecs update-service --cluster $ECS_CLUSTER \
                    --service $ECS_SERVICE \
                    --force-new-deployment \
                    --region $AWS_REGION
                    '''
                }
            }
        }
    }
}
