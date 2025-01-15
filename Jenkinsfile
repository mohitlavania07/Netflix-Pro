pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'netflix'
        DOCKER_REPO = 'mohitlavania07'
        API_KEY = '2990d632bc60ed57ad3d8a4e109fca24'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Cloning Code From GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/mohitlavania07/Netflix-Pro.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('OWASP Dependency Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerPass', usernameVariable: 'dockerUser')]) {
                        sh '''
                            docker login -u ${dockerUser} -p ${dockerPass}
                            docker build --build-arg TMDB_V3_API_KEY=${API_KEY} -t ${DOCKER_IMAGE} .
                            docker tag ${DOCKER_IMAGE} ${dockerUser}/${DOCKER_IMAGE}:latest
                            docker push ${dockerUser}/${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image mohitlavania07/netflix:latest > trivyimage.txt'
            }
        }
        stage('Deploy to Docker Container') {
            steps {
                sh 'docker run -d --name netflix -p 8081:80 mohitlavania07/netflix:latest'
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                    Project: ${env.JOB_NAME}<br/>
                    Build Number: ${env.BUILD_NUMBER}<br/>
                    URL: ${env.BUILD_URL}<br/>
                """,
                to: 'mohitlavania2003@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
