pipeline {
  agent any
  environment {
      repo_name="${MicroServices}"
      release_name="${Release}"
      release_type="${Release-Type}"
  }

  stages {
    stage('Git Pull') {
        steps {
            //script {branch_name=releaseName(release_name)}
            if ("$release_name" == 'Stable') {
              branch_name=stable
            } else if ("Alpha".equals(${release_name})) {
              branch_name=development
            } else if ("Stable".equals($release_name)) {
              branch_name=pre-stable
            }
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
def releaseName(release) {
    if ("Stable".equals(release)) {
        branch_name=stable
      } else if ("Alpha".equals(release)) {
        branch_name=development
      } else if ("Stable".equals(release)) {
        branch_name=pre-stable
      }
}