
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'manya23r/my-python-app3'
        CONTAINER_NAME = 'my-python-container'
    }

    stages {

        stage('Docker Environment Check') {
            steps {
                powershell '''
                    Write-Host "Checking Docker..."
                    docker --version
                    docker info
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:latest .'
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
                    powershell '''
                        $token = $env:DOCKER_TOKEN.Trim()

                        if ([string]::IsNullOrWhiteSpace($token)) {
                            throw "Docker Hub token is empty"
                        }

                        Write-Host "Logging in as $env:DOCKER_USER"

                        $bytes = [System.Text.Encoding]::UTF8.GetBytes($token + "`n")

                        $psi = New-Object System.Diagnostics.ProcessStartInfo
                        $psi.FileName = "docker"
                        $psi.Arguments = "login -u $env:DOCKER_USER --password-stdin"
                        $psi.UseShellExecute = $false
                        $psi.RedirectStandardInput = $true
                        $psi.RedirectStandardOutput = $true
                        $psi.RedirectStandardError = $true

                        $process = New-Object System.Diagnostics.Process
                        $process.StartInfo = $psi
                        [void]$process.Start()

                        $stdoutTask = $process.StandardOutput.ReadToEndAsync()
                        $stderrTask = $process.StandardError.ReadToEndAsync()

                        $process.StandardInput.BaseStream.Write(
                            $bytes, 0, $bytes.Length
                        )
                        $process.StandardInput.Close()

                        $process.WaitForExit()

                        $output = $stdoutTask.Result
                        $errorOutput = $stderrTask.Result

                        Write-Host $output

                        if ($process.ExitCode -ne 0) {
                            Write-Host $errorOutput
                            throw "Docker Hub login failed"
                        }

                        Write-Host "Docker Hub login successful"
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%:latest'
            }
        }

        stage('Deploy Container') {
            steps {
                powershell '''
                    $existing = docker ps -aq --filter "name=^my-python-container$"

                    if ($existing) {
                        docker stop my-python-container
                        docker rm my-python-container
                    }

                    docker run -d `
                        --name my-python-container `
                        -p 5000:5000 `
                        manya23r/my-python-app3:latest

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo 'Application URL: http://localhost:5000'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }
    }
}
