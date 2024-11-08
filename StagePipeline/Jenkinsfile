pipeline {
    agent any

    environment {
        // AWS ECR repository URL
        ECR_REPOSITORY = 'vprofileapp'
        AWS_REGION = 'us-east-1' // Change to your region
        IMAGE_TAG = "${BUILD_NUMBER}"  // Use Jenkins build number as the tag
        IMAGE_LATEST_TAG = "latest"
        AWS_ACCOUNT_ID = '730335281764'
        DOCKERFILE_DIR = './Docker-files/app/multistage'  // Directory where the Dockerfile is located
        cluster = "javastaging"
        service = "javaappsvc"
    }

    stages {
        

        stage('Login to AWS ECR') {
            steps {
                script {
                    // Use the stored AWS credentials from Jenkins Credentials Plugin
                    withAWS(credentials: 'awscreds'){
                        // Authenticate to AWS ECR using the AWS CLI
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    //sh "docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} ."
                    //sh "docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} -f ./Docker-files/app/multistage/Dockerfile"
                    //sh "docker buildx build -t ${ECR_REPOSITORY}:${IMAGE_TAG} -f ./Docker-files/app/multistage/Dockerfile"
                    sh """
                        docker buildx build -t ${ECR_REPOSITORY}:${IMAGE_TAG} -f ${DOCKERFILE_DIR}/Dockerfile ${DOCKERFILE_DIR}
                    """


                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag the image for ECR
                    sh "docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_LATEST_TAG}"
                    //docker tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_LATEST_TAG}
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the image to AWS ECR
                    sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_LATEST_TAG}"
                }
            }
        }
        stage("Deploy to ECS staging"){
            steps{
                withAWS(credentials: 'awscreds', region: 'us-east-1'){
                    sh 'aws ecs update-service --cluster ${cluster} --service ${service}  --force-new-deployment'
                }
            }
        }
    }

    post {
        success {
            echo "Docker image pushed to ECR successfully!"
        }
        failure {
            echo "Build or push failed!"
        }
    }
}
