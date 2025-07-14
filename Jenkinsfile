pipeline {
    agent any

    tools {
        dotnet 'dotnet8'
    }

    stages {

        stage('Clone') {
            steps {
                echo 'Cloning WebBanHangOnline source...'
                git branch: 'master', url: 'https://github.com/SpringMuch/WebBanHangOnline.git'
            }
        }

        stage('Restore') {
            steps {
                echo 'Restoring packages...'
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
                bat 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'dotnet test --no-build --verbosity normal'
            }
        }

        stage('Publish') {
            steps {
                echo 'Publishing to ./publish...'
                bat 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Copy to IIS folder') {
            steps {
                echo 'Copying to C:\\wwwroot\\WebBanHangOnline...'
                bat 'xcopy "%WORKSPACE%\\publish" "C:\\wwwroot\\WebBanHangOnline" /E /Y /I /R'
            }
        }

        stage('Deploy to IIS') {
            steps {
                echo 'Deploying to IIS...'
                powershell '''
                    Import-Module WebAdministration
                    if (-not (Test-Path IIS:\\Sites\\WebBanHangOnline)) {
                        New-Website -Name "WebBanHangOnline" -Port 81 -PhysicalPath "C:\\wwwroot\\WebBanHangOnline"
                    } else {
                        Restart-WebAppPool -Name "DefaultAppPool"
                    }
                '''
            }
        }

    } // end stages
}
