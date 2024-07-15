pipeline {
  agent any
  environment {
      repo_name="${MicroServices}"
      branch_name="${Release}"
      release_type="${Release-Type}"
  }

  stages {
    stage('Git Clone') {
        steps {
          script {
            if ("${repo_name}" == "Core") {
              dir('acs-connector') {
                git(url: 'https://git.assistanz.com/stackbill/acs-connector.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              dir('wolf') {
                git(url: 'https://git.assistanz.com/stackbill/wolf.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              sh 'mv ./wolf/Dockerfile Dockerfile && mv ./wolf/.dockerignore .dockerignore'
              sh 'ls -al'
            }
            if ("${repo_name}" == "Billing") {
              dir('acs-connector') {
                git(url: 'https://git.assistanz.com/stackbill/acs-connector.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              dir('wolf') {
                git(url: 'https://git.assistanz.com/stackbill/Odolf.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              sh 'mv ./wolf/Dockerfile Dockerfile && mv ./wolf/.dockerignore .dockerignore'
              sh 'ls -al'
            }
          }
        }
    }
  }
  post { 
        success { 
            cleanWs()
        }
    }
}