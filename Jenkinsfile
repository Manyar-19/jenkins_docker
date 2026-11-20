
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'naveen04jan/my-python-app3'
        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'my-python-container'
        APP_PORT = '5000'
    }

    stages {

        stage('Docker Environment Check') {
            steps {
                powershell '''
                    Write-Host "Checking Docker..."
                    whoami
                    docker --version
                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-login',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
                        powershell '''
                            $ErrorActionPreference = "Stop"

                            New-Item -ItemType Directory -Force -Path $env:DOCKER_CONFIG | Out-Null

                            $env:DOCKER_TOKEN.Trim() | docker --config $env:DOCKER_CONFIG login -u $env:DOCKER_USER --password-stdin

                            if ($LASTEXITCODE -ne 0) {
                                throw "DockerHub login failed"
                            }

                            Write-Host "DockerHub login successful"
                        '''
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
                    bat 'docker push %DOCKER_IMAGE%:%IMAGE_TAG%'
                }
            }
        }

        stage('Deploy Container') {
            steps {
                withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
                    bat '''
                        docker stop %CONTAINER_NAME% 2>NUL
                        docker rm %CONTAINER_NAME% 2>NUL

                        docker run -d ^
                            --name %CONTAINER_NAME% ^
                            -p %APP_PORT%:%APP_PORT% ^
                            %DOCKER_IMAGE%:%IMAGE_TAG%

                        docker ps
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo 'Docker image pushed and application deployed.'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }

        always {
            bat '''
                if exist "%WORKSPACE%\\.docker" (
                    rmdir /s /q "%WORKSPACE%\\.docker"
                )
            '''
            echo 'Pipeline execution completed.'
        }
    }
}
