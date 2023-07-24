pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '7'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/test-jenkins']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CleanCheckout']],
                    userRemoteConfigs: [[credentialsId: 'pokhmel_github_test_tools', url: 'git@github.com:po-khmel/usegalaxy-it-tools.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                deleteDir()
                withCredentials([file(credentialsId: 'USEGALAXY_IT_API', variable: 'API_KEY')]) {
                    withPythonEnv('Python-3.8.10') {
                        sh 'pip install -r requirements.txt'
                        sh 'make fix'
                        sh 'make install GALAXY_SERVER=https://usegalaxy-it.ext.cineca.it GALAXY_API_KEY=${API_KEY}'
                        sh 'git add *.yaml.lock'
                        sh 'git commit -m "Update lock files. Jenkins Build: ${BUILD_NUMBER}" -m "https://github.com/po-khmel/usegalaxy-it-tools/blob/test-jenkins/reports/$(date +%Y-%m-%d-%H-%M)-tool-update.md" || true'
                    }
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'report.log', onlyIfSuccessful: true
            git branch: 'test-jenkins', credentialsId: 'pokhmel_github_test_tools', pushOnlyIfSuccess: true, url: 'git@github.com:po-khmel/usegalaxy-it-tools.git'
        }
        always {
            emailext body: '', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult}", to: 'khmelevskayapv@gmail.com', sendToIndividuals: true, unstable: true
        }
    }
}
