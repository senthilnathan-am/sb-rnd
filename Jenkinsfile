pipeline {
  agent any
  environment {
      repo_name="${MicroServices}"
      release_name="${Release}"
      release_type="${Release-Type}"
      //branch_name=stable
      branch_name="stable"
  }

  stages {
    stage('Git Pull') {
        steps {
            dir('acs-connector') {
              git(url: 'https://git.assistanz.com/stackbill/acs-connector.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
            }
            dir('wolf') {
              git(url: 'https://git.assistanz.com/stackbill/wolf.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
            }
            sh 'mv ./wolf/Dockerfile Dockerfile && mv ./wolf/.dockerignore .dockerignore'
            sh 'ls -al'
        }
    }
  }
}