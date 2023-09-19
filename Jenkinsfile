//
// This is an example of using VeraDemo Java test application with the Veracode Static scanner.  
//

pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'Verademo'      // App Name in the Veracode Platform
    }

    stages{
        stage ('environment verify') {
            steps {
                script {
                    if (isUnix() == true) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                    }
                    else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }

        // this assumes you have the `dotnet` command-line tools installed and available to Jenkins
        stage ('build') {
            steps {
                script {
                    if(isUnix() == true) {
                        sh 'dotnet publish -c Debug -p:UseAppHost=false'
                    }
                    else {
                        bat 'dotnet publish -c Debug -p:UseAppHost=false'
                    }
                }
            }
        }

        stage ('Veracode scan') {
            // zip archive for Veracode scanning. Uses `zip` from the Pipline Utility Steps plugin
            zip zipFile: 'upload.zip', archive: false, glob: 'app/bin/Debug/netcoreapp3.1/**'

            steps {
                script {
                    if(isUnix() == true) {
                        env.HOST_OS = 'Unix'
                    }
                    else {
                        env.HOST_OS = 'Windows'
                    }
                }

                echo 'Veracode scanning'
                withCredentials([ usernamePassword ( 
                    credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_ID', passwordVariable: 'VERACODE_API_KEY') ]) {
                        // fire-and-forget 
                        veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, fileNamePattern: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: 'Jenkins-${BUILD_NUMBER}', uploadExcludesPattern: '', uploadIncludesPattern: 'upload.zip', vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"

                        // wait for scan to complete (timeout: x)
                        //veracode applicationName: '${VERACODE_APP_NAME}'', criticality: 'VeryHigh', debug: true, timeout: 20, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}", uploadExcludesPattern: '', uploadIncludesPattern: 'target/verademo.war', vid: '${VERACODE_API_ID}', vkey: '${VERACODE_API_KEY}'
                    }      
            }
        }

// the above steps are the bare minimum.
// below are some additional steps that are commonplace

        stage ('Veracode SCA') {
            steps {
                echo 'Veracode SCA'
                withCredentials([ string(credentialsId: 'SCA_Token', variable: 'SRCCLR_API_TOKEN')]) {
                    withMaven(maven:'maven-3') {
                        script {
                            if(isUnix() == true) {
                                sh "curl -sSL https://download.sourceclear.com/ci.sh | sh -s -- scan app"

                                // debug, no upload
                                //sh "curl -sSL https://download.sourceclear.com/ci.sh | DEBUG=1 sh -s -- scan --no-upload"
                            }
                            else {
                                powershell '''
                                            Set-ExecutionPolicy AllSigned -Scope Process -Force
                                            $ProgressPreference = "silentlyContinue"
                                            iex ((New-Object System.Net.WebClient).DownloadString('https://download.srcclr.com/ci.ps1'))
                                            srcclr scan app
                                            '''
                            }
                        }
                    }
                }
            }
        }

        // Currently only works on *nix
        stage ('Veracode container scan') {
            steps {
                echo 'Veracode container scanning'
                withCredentials([ usernamePassword ( 
                    credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_KEY_ID', passwordVariable: 'VERACODE_API_KEY_SECRET') ]) {
                        script {
                            if(isUnix() == true) {
                                sh '''
                                    curl -fsS https://tools.veracode.com/veracode-cli/install | sh
                                    ./veracode scan --type directory --source . --format table
                                    '''
                            }
                        }
                    }
            }
        }
    }
}
