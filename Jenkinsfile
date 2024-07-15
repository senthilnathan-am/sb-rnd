pipeline {
  agent any
  environment {
      repo_name="${MicroServices}"
      release_name="${Release}"
      if ( "$release_name" == Stable ) {
        branch_name=stable
      }
      if ( "$release_name" == Alpha ) {
        branch_name=development
      }
      if ( "$release_name" == Stable ) {
        branch_name=pre-stable
      }  
      release_type="${Release-Type}"
      COSIGN_PASSWORD=credentials('78eadf7d-e1af-4058-ab98-d17f2a54839c')
  }

  stages {
    stage('Git Pull') {
        steps {
            sh 'find . -type f -delete'
            dir('acs-connector') {
              git(url: 'https://git.assistanz.com/stackbill/acs-connector.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: '$branch_name')
            }
            dir('wolf') {
              git(url: 'https://git.assistanz.com/stackbill/wolf.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: '$branch_name')
            }
            sh 'mv ./wolf/Dockerfile Dockerfile && mv ./wolf/.dockerignore .dockerignore'
            sh 'ls -al'
        }
    }
  }
}