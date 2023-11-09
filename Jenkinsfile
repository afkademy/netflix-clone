pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/afkademy/netflix-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
                archiveArtifacts artifacts: 'trivyfs.txt'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){ 
						withCredentials([string(credentialsId: 'tmdb_api_key', variable: 'TMDB_API_KEY')]) {
							sh "docker build --build-arg TMDB_V3_API_KEY=${TMDB_API_KEY} -t netflix ."
						}
                       sh "docker tag netflix kinason/netflix:latest "
                       sh "docker push kinason/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image kinason/netflix:latest > trivyimage.txt" 
                archiveArtifacts artifacts: 'trivyimage.txt'
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d -p 8081:80 kinason/netflix:latest'
            }
        }
    }
}

