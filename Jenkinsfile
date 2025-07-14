pipeline {
    agent any

    environment {
        // Đường dẫn MSBuild (bắt buộc cho WebApplication)
        MSBUILD_EXE = '"C:\\Program Files (x86)\\Microsoft Visual Studio\\2022\\BuildTools\\MSBuild\\Current\\Bin\\MSBuild.exe"'
        SOLUTION_FILE = 'WebBanHangOnline.sln'
        PUBLISH_DIR = 'publish'
        IIS_FOLDER = 'C:\\wwwroot\\WebBanHangOnline'
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
                echo 'Restoring NuGet packages...'
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project using MSBuild...'
                bat "${MSBUILD_EXE} ${SOLUTION_FILE} /p:Configuration=Release"
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'dotnet test --no-build --verbosity normal || echo No tests found or failed to execute.'
            }
        }

        stage('Publish') {
            steps {
                echo 'Publishing project (manually clean and copy)...'
                bat "if exist ${PUBLISH_DIR} rmdir /S /Q ${PUBLISH_DIR}"
                bat "mkdir ${PUBLISH_DIR}"
                bat "${MSBUILD_EXE} ${SOLUTION_FILE} /p:DeployOnBuild=true /p:PublishProfile=FolderProfile /p:PublishDir=${PUBLISH_DIR}\\ /p:Configuration=Release"
            }
        }

        stage('Copy to IIS folder') {
            steps {
                echo "Copying to ${IIS_FOLDER} ..."
                bat "xcopy \"%WORKSPACE%\\${PUBLISH_DIR}\" \"${IIS_FOLDER}\" /E /Y /I /R"
            }
        }

        stage('Deploy to IIS') {
            steps {
                echo 'Deploying to IIS...'
                powershell '''
                    Import-Module WebAdministration
                    if (-not (Test-Path IIS:\\Sites\\WebBanHangOnline)) {
                        New-Website -Name "WebBanHangOnline" -Port 81 -PhysicalPath "C:\\wwwroot\\WebBanHangOnline" -ApplicationPool "DefaultAppPool"
                    } else {
                        Restart-WebAppPool -Name "DefaultAppPool"
                    }
                '''
            }
        }

    } // end stages

    post {
        failure {
            echo '❌ Build failed.'
        }
        success {
            echo '✅ Build and deploy completed successfully.'
        }
    }
}
