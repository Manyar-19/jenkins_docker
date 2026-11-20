// Stage 3: Login to Docker Hub
        stage('Login to DockerHub') {
            steps {
                withEnv(["DOCKER_CONFIG=${env.WORKSPACE}\\.docker"]) {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-login',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_TOKEN'
                        )
                    ]) {
                        powershell '''
                            $ErrorActionPreference = "Stop"

                            New-Item -ItemType Directory `
                                -Force -Path $env:DOCKER_CONFIG | Out-Null

                            $configPath = Join-Path $env:DOCKER_CONFIG "config.json"
                            '{"auths":{}}' | Set-Content -Path $configPath -Encoding ASCII

                            $cleanUser = $env:DOCKER_USER.Trim()
                            $cleanToken = $env:DOCKER_TOKEN.Trim()

                            if ([string]::IsNullOrWhiteSpace($cleanUser) -or [string]::IsNullOrWhiteSpace($cleanToken)) {
                                throw "DOCKER_USER or DOCKER_TOKEN is empty"
                            }

                            Write-Host "Logging in as user length: $($cleanUser.Length)"

                            $psi = New-Object System.Diagnostics.ProcessStartInfo
                            $psi.FileName = "docker.exe"
                            $psi.Arguments = "--config `"$env:DOCKER_CONFIG`" login -u `"$cleanUser`" --password-stdin"
                            $psi.UseShellExecute = $false
                            $psi.RedirectStandardInput = $true
                            $psi.RedirectStandardOutput = $true
                            $psi.RedirectStandardError = $true
                            $psi.CreateNoWindow = $true

                            $p = New-Object System.Diagnostics.Process
                            $p.StartInfo = $psi

                            [void]$p.Start()

                            $bytes = [System.Text.Encoding]::ASCII.GetBytes($cleanToken + "`n")
                            $p.StandardInput.BaseStream.Write($bytes, 0, $bytes.Length)
                            $p.StandardInput.Close()

                            $output = $p.StandardOutput.ReadToEnd()
                            $errorOutput = $p.StandardError.ReadToEnd()
                            $p.WaitForExit()

                            Write-Host $output

                            if ($p.ExitCode -ne 0) {
                                Write-Host $errorOutput
                                throw "DockerHub login failed"
                            }

                            Write-Host "DockerHub login successful"
                        '''
                    }
                }
            }
        }
