pipeline {
    agent any
    environment {
        DOCKERHUB_CRED_ID = 'mlops-docker-token'
        DOCKERHUB_REGISTRY = 'https://registry.hub.docker.com'
        DOCKERHUB_REPOSITORY = 'mehmeh8/py-project' 
        /* DOCKER_CONTEXT=default */
    }
    stages {
        stage('Clone Repository') {
            steps {
                // Clone Repository
                script {
                    echo 'Cloning GitHub Repository...'
                    //checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '55aa2c97-2046-40e5-ade4-2570a0e5080f', url: 'https://github.com/mehardeep88/mlops.git']])
                    checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'mlops-git-token', url: 'https://github.com/mehardeep88/mlops.git']])
                    //checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'mlops', url: 'https://github.com/mehardeep88/mlops.git']])
                }
            }
        }
        stage('Lint Code') {
            steps {
                // Lint code
                script {
                    echo 'Linting Python Code...'
                    sh "python --version"
                    sh "python -m pip install --break-system-packages -r requirements.txt"
                    sh "pylint app.py train.py --output=pylint-report.txt --exit-zero"
                    sh "flake8 app.py train.py --ignore=E501,E302 --output-file=flake8-report.txt || true"
                    sh "black app.py train.py || true"
                    //bat "python -m pip install -r requirements.txt"
                    //bat "python -m pip install --break-system-packages -r requirements.txt"
                    //bat "pylint app.py train.py --output=pylint-report.txt --exit-zero"
                    //bat "python -m flake8 app.py train.py --ignore=E501,E302 --output-file=flake8-report.txt || exit 0"
                    //bat "black app.py train.py"
                }
            }
        }
        stage('Test Code') {
            steps {
                // Pytest code
                script {
                    echo 'Testing Python Code...'
                    sh "pytest tests/"
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                script {
                    echo 'Starting Trivy file system scan...'
                    //bat 'trivy -v'
                    //sh 'trivy fs --exit-code 0 --skip-dirs .venv --skip-files pylint-report.txt,flake8-report.txt,black-report.txt .'
                    sh 'trivy fs ./ --format table -o trivy-fs-report.html'
                }
            }
        }
        stage('Build Docker Image') {
            /* environment {
                 DOCKER_BUILDKIT = '1'
            } */
            steps {
                // Build Docker Image
                script {
                    echo 'Building Docker Image...'
                    //bat "wsl docker buildx build --builder default -t ${DOCKERHUB_REPOSITORY}:latest ." 
                    /* def dockerImage = docker.build("${DOCKERHUB_REPOSITORY}:latest")    */           
                    //dockerImage = docker.build("mlops") 
                    dockerImage = docker.build("${DOCKERHUB_REPOSITORY}:latest") 
                }
            }
        }
        stage('Trivy Docker Image Scan') {
            steps {
                // Trivy Docker Image Scan
                script {
                    echo 'Scanning Docker Image with Trivy...'
                    sh "trivy image ${DOCKERHUB_REPOSITORY}:latest --format table -o trivy-image-report.html"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                // Push Docker Image to DockerHub
                script {
                    echo 'Pushing Docker Image to DockerHub...'
                //withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIAL_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                //sh '''
                //echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                //docker push mehmeh8/py-project:latest
                //'''
                    docker.withRegistry("${DOCKERHUB_REGISTRY}","${DOCKERHUB_CRED_ID}"){
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                //deploy to aws
                script {
                    echo 'Deploying to production...'
                     sh "aws ecs update-service --cluster ml-ecs --service mlops-ecs-task-def-service --region us-east-1 --force-new-deployment"
                }
            }
        }
    }
}