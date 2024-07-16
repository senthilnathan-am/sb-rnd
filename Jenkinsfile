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
              dir('odolf') {
                git(url: 'https://git.assistanz.com/stackbill/Odolf.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              sh 'mv ./odolf/Dockerfile Dockerfile && mv ./odolf/.dockerignore .dockerignore'
              sh 'ls -al'
            }
          }
        }
    }

  stage('Maven Build') {
        steps {
            /** Maven Build **/
          script {
            if ("${repo_name}" == "Core") {
              sh '''
                #update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/bin/java
                sudo unlink /etc/alternatives/java && sudo ln -sf /usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/java /etc/alternatives/java
                java -version && mvn -version
                cd ./acs-connector/ && mvn clean install -U -DskipTests=true && cd ..
                cp ./acs-connector/target/connectors-1.0.0-SNAPSHOT.jar ./wolf
                cd ./wolf && mvn clean package -DskipTests=true
              '''
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