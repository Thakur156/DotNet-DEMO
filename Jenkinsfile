pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git 'https://github.com/Thakur156/DotNet-DEMO'
            }
        }
        
        
       stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=dotnet-demo \
                    -Dsonar.projectKey=dotnet-demo """
                }
            }
        }
        
        stage('Docker build & tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "make image"
                    }
                } 
            }
        }
        
        stage('Trivy image Scan') {
            steps {
                sh "trivy image thakur156/dotnet-demoapp:latest "
            }
        }
        
        stage('Docker Push') {
            steps {
                sh "make push "
            }
        }
        
        stage('Run Container') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "docker run -d -p 5000:5000 thakur156/dotnet-demoapp:latest "
                }
            }
        }
    }
}
