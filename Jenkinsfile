pipeline {
    agent any

    environment {
        DOCKER_USERNAME = 'vinhbh'
        DOCKER_HUB_CREDENTIALS = 'dockerhub-acc'
        IMAGE_NAME = "${DOCKER_USERNAME}/flask-sum-api:2.0"
        CONTAINER_NAME = "flask-test-container"
    }

    stages {
        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh "docker run --rm ${IMAGE_NAME} python -m unittest discover -s /app -p 'test_*.py'"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-acc', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                }
                sh "docker push ${IMAGE_NAME}"
            }
        }

        stage('Deploy web server') {
            steps {
                ansiblePlaybook credentialsId: 'ansible2', 
                disableHostKeyChecking: true, 
                installation: 'ansible2', 
                inventory: './ansible/inventory', 
                playbook: './ansible/playbooks/ansible.yml', 
                vaultTmpPath: ''
                // extras: "-t api_server -e DJANGO_IMAGE_VERSION=${env.TAG_NAME}"
            }
        }
    }

    post {
        always {
            echo "Job Completed! Cleaning up resources..."
        }

        success {
            echo "Build & Test Passed. Image pushed to Docker Hub successfully!"
        }

        failure {
            echo "Job failed. Cleaning up unused Docker resources..."
        }
    }
}
