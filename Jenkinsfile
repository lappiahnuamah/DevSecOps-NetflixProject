pipeline {
    agent any

    tools {
        jdk 'java-17'
        nodejs 'recent node'
    }
    environment {
        SONAR_TOKEN = credentials('sonar-token') // Replace with your SonarQube token ID in Jenkins
        // DOCKERHUB_CREDENTIALS = credentials('dockerhub-id') // Replace with your DockerHub credentials ID
        IMAGE_NAME = "lappiahnuamah/devsecops-netflixproject"
        SCANNER_HOME = tool 'sonar-scanner'
        NVD_API_KEY = credentials('nvd-api-key')
        REGISTRY = "docker.io"
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/lappiahnuamah/DevSecOps-NetflixProject'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=DevSecOps-NetflixProject \
                        -Dsonar.projectKey=DevSecOps-NetflixProject '''
                    }
                }
            }
        }

        stage('quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }


       stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--nvdApiKey $NVD_API_KEY --scan ./ --disableYarnAudit --disableNodeAudit',
                odcInstallation: 'DP-Check'
            }
        }


       stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/']){
                       sh "docker build --build-arg TMDB_V3_API_KEY=26df1f47263a985c0513b5e8e08a23a7 -t netflix ."
                       sh "docker tag netflix $REGISTRY/$IMAGE_NAME"
                       sh "docker push $REGISTRY/$IMAGE_NAME"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image $IMAGE_NAME > trivyimage.txt" 
            }
        }

        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflixclone -p 8081:80 $REGISTRY/$IMAGE_NAME'
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
