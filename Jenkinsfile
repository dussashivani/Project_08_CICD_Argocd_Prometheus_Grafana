pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Cloning GitHub repository'

                git branch: 'main',
                    url: 'https://github.com/dussashivani/Project_08_CICD_Argocd_Prometheus_Grafana.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        def scannerHome = tool 'SonarScannerCLI'

                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=my-devops-app \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://44.192.107.94:9000 \
                            -Dsonar.login=squ_c196ecd2a0134d34f72c83fffc0c91beea7f8ae7
                        """
                    }
                }
            }
        }

        stage('Building the code') {
            steps {
                echo 'Installing Node.js dependencies'

                sh 'ls -ltr'

                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'

                sh '''
                    sudo docker build \
                    -t shivanidussa/nodejs:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Docker Image Scan') {
            steps {
                echo 'Scanning Docker image with Trivy'

                sh """
                    sudo trivy image shivanidussa/nodejs:${BUILD_NUMBER}
                """
            }
        }

        stage('Push Image to ECR') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'aws-access-key',
                        variable: 'AWS_ACCESS_KEY_ID'
                    ),
                    string(
                        credentialsId: 'aws-secret-key',
                        variable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {

                    sh '''
                        export AWS_DEFAULT_REGION=us-east-1

                        echo "Logging in to Amazon ECR..."

                        aws ecr get-login-password --region us-east-1 \
                        | sudo docker login \
                        --username AWS \
                        --password-stdin \
                        518216637461.dkr.ecr.us-east-1.amazonaws.com

                        echo "Tagging Docker image..."

                        sudo docker tag \
                        shivanidussa/nodejs:${BUILD_NUMBER} \
                        518216637461.dkr.ecr.us-east-1.amazonaws.com/nodejs:${BUILD_NUMBER}

                        echo "Pushing Docker image to ECR..."

                        sudo docker push \
                        518216637461.dkr.ecr.us-east-1.amazonaws.com/nodejs:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Update Deployment File') {

            environment {
                GIT_REPO_NAME = "Project_08_CICD_Argocd_Prometheus_Grafana"
                GIT_USER_NAME = "dussashivani"
            }

            steps {

                echo 'Updating Kubernetes deployment image'

                withCredentials([
                    string(
                        credentialsId: 'github-token',
                        variable: 'githubtoken'
                    )
                ]) {

                    sh '''
                        git config user.email "shivanidussa@gmail.com"

                        git config user.name "shivani"

                        echo "Updating deployment.yml..."

                        sed -i "s|image: .*|image: 518216637461.dkr.ecr.us-east-1.amazonaws.com/nodejs:${BUILD_NUMBER}|" deploymentfiles/deployment.yml

                        echo "Checking updated deployment.yml..."

                        cat deploymentfiles/deployment.yml

                        git add deploymentfiles/deployment.yml

                        git commit \
                        -m "Update deployment image to version ${BUILD_NUMBER}"

                        git push \
                        https://${githubtoken}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git \
                        HEAD:main
                    '''
                }
            }
        }
    }
}
