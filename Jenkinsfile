pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIAL_ID = 'DOCKERHUB_CREDENTIAL_ID'
        DOCKERHUB_REGISTRY = 'https://index.docker.io/v1/'
        DOCKERHUB_REPOSITORY = 'mehmeh8/py-project' 
    }
    stages {
        stage('Clone Repository') {
            steps {
                // Clone Repository
                script {
                    echo 'Cloning GitHub Repository...'
                    checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '55aa2c97-2046-40e5-ade4-2570a0e5080f', url: 'https://github.com/mehardeep88/mlops.git']])
                    //checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'mlops', url: 'https://github.com/mehardeep88/mlops.git']])
                }
            }
        }
        stage('Lint Code') {
            steps {
                // Lint code
                script {
                    echo 'Linting Python Code...'
                    //bat "python -m pip install -r requirements.txt"
                    bat "python -m pip install --break-system-packages -r requirements.txt"
                    bat "pylint app.py train.py --output=pylint-report.txt --exit-zero"
                    bat "python -m flake8 app.py train.py --ignore=E501,E302 --output-file=flake8-report.txt || exit 0"
                    bat "black app.py train.py"
                }
            }
        }
        stage('Test Code') {
            steps {
                // Pytest code
                script {
                    echo 'Testing Python Code...'
                    bat "pytest tests/"
                }
            }
        }
        stage('Check Docker') {
            steps {
                script {
                    bat 'docker --version'
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                // Trivy Filesystem Scan
                script {
                    echo 'Scannning Filesystem with Trivy...'
                    bat 'docker run --rm -v %cd%:/project aquasec/trivy fs /project --format table -o /project/trivy-fs-report.html'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                // Build Docker Image
                script {
                    echo 'Building Docker Image...'
                    /* dockerImage = docker.build("${DOCKERHUB_REPOSITORY}:latest")  */
                }
            }
        }
        stage('Trivy Docker Image Scan') {
            steps {
                // Trivy Docker Image Scan
                script {
                    echo 'Scanning Docker Image with Trivy...'
                    /* bat "trivy image ${DOCKERHUB_REPOSITORY}:latest --format table -o trivy-image-report.html" */
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                // Push Docker Image to DockerHub
                script {
                    echo 'Pushing Docker Image to DockerHub...'
                    /* docker.withRegistry("${DOCKERHUB_REGISTRY}", "${DOCKERHUB_CREDENTIAL_ID}"){
                        dockerImage.push('latest') */
                    }
                }
            }
        
        stage('Deploy') {
            steps {
                // Deploy Image to Amazon ECS
                script {
                    echo 'Deploying to production...'
                        /* sh '''
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                        az aks get-credentials --resource-group <your-resource-group> --name <your-aks-cluster>
                        kubectl set image deployment/<your-deployment-name> <container-name>=<yourimage>:<tag>
                    ''' */
                    }
                }
            }
        }
}