pipeline {
    agent any 

    // Define variables for easy updates
    environment {
        DOCKER_REPO = "vaishu09/secure-hybrid"
        IMAGE_TAG = "v1"
    }

    stages {
        // Stage 1: Pull the latest code from GitHub
        stage('Pull Code') {
            steps {
                checkout scm
                echo 'Successfully pulled code from GitHub!'
            }
        }

        // Stage 2: Build the Docker Image on VM2
        stage('Build') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t ${DOCKER_REPO}:${IMAGE_TAG} ."
            }
        }

        // Stage 3: Security Scan (The "Gatekeeper")
        // We scan BEFORE pushing to ensure only secure images are stored/deployed
        stage('Trivy Security Scan') {
            steps {
                echo 'Starting Security Scan with Trivy...'
                // --exit-code 1 ensures the pipeline FAILS if a CRITICAL bug is found
                sh "trivy image --severity CRITICAL --exit-code 1 --timeout 15m ${DOCKER_REPO}:${IMAGE_TAG}"


            }
        }

        // Stage 4: Push to Docker Hub
        // This only happens if the Trivy scan passes
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'docker_hub_creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_REPO}:${IMAGE_TAG}"
                }
            }
        }

        // Stage 5: Remote Deploy to K8s on VM3
        stage('Deploy to K8s') {
            steps {
                echo 'Deploying to Kubernetes on VM3...'
                // Applies your YAML (Ensure this file is in your GitHub repo)
                sh 'kubectl apply -f project/project-deploy.yaml'
                
                // Forces K8s to pull the fresh image we just pushed to the Hub
                sh 'kubectl rollout restart deployment/secure-app-deployment'
                
                echo 'Deployment Complete!'
            }
        }
    }

    // Post-build actions for reporting
    post {
        success {
            echo "Pipeline Successful! Access app at: http://192.168.30.2"
        }
        failure {
            echo "Pipeline Failed! Check Trivy logs for security vulnerabilities or check Docker credentials."
        }
    }
}
