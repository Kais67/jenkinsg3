pipeline {
    agent any
    
    options {
        timestamps()
    }
    
    environment {
        IMAGE = 'kais67/monapp'
        TAG = "build-${env.BUILD_NUMBER}"
        DOCKER_PATH = 'C:\\Users\\kaisc\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
    }
    
    stages {
        stage('Checkout') {
            steps { 
                checkout scm 
            }
        }
        
        stage('Docker Build') {
            steps {
                bat "\"%DOCKER_PATH%\" version"
                bat "\"%DOCKER_PATH%\" build -t %IMAGE%:%TAG% ."
            }
        }
        
        stage('Smoke Test') {
            steps {
                bat """
                    "%DOCKER_PATH%" rm -f monapp_test 2>nul || ver > nul
                    "%DOCKER_PATH%" run -d --name monapp_test -p 8082:80 %IMAGE%:%TAG%
                    C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe -Command "Start-Sleep -Seconds 3; if ((Invoke-WebRequest -Uri 'http://localhost:8082' -UseBasicParsing).StatusCode -eq 200) { exit 0 } else { exit 1 }"
                    "%DOCKER_PATH%" rm -f monapp_test
                """
            }
        }
        
        stage('Push (Docker Hub)') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat """
                        "%DOCKER_PATH%" login -u %USER% -p %PASS%
                        "%DOCKER_PATH%" tag %IMAGE%:%TAG% %IMAGE%:latest
                        "%DOCKER_PATH%" push %IMAGE%:latest
                    """
                }
            }
        }
    }
    
    post {
        success { echo 'Build+Test+Push OK' }
        failure { echo 'Build/Tests/Push KO' }
    }
}