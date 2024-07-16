pipeline {
  agent any
  environment {
      repo_name="${MicroServices}"
      branch_name="${Release}"
      release_type="${Release-Type}"
      COSIGN_PASSWORD=credentials('11aad1b0-d788-4813-ab4a-81fd4bbb7487')
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
                sudo update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/bin/java
                java -version && mvn -version
                cd ./acs-connector/ && mvn clean install -U -DskipTests=true && cd ..
                cp ./acs-connector/target/connectors-1.0.0-SNAPSHOT.jar ./wolf
                cd ./wolf && mvn clean package -DskipTests=true
              '''
            }
          }
        }
    }

    stage('Signing Artifact') {
        steps {
            script {
                if(fileExists('/var/lib/jenkins/cosign/keys/artf.key')) {
                    sh 'COSIGN_PASSWORD=$COSIGN_PASSWORD cosign sign-blob --key="/var/lib/jenkins/cosign/keys/artf.key" --output="/var/lib/jenkins/cosign/sign-files/artf" -y ./wolf/target/wolf-0.0.1-SNAPSHOT.jar'
                }
                else {
                    sh 'COSIGN_PASSWORD=$COSIGN_PASSWORD cosign generate-key-pair --output-key-prefix="/var/lib/jenkins/cosign/keys/artf"'
                    sh 'COSIGN_PASSWORD=$COSIGN_PASSWORD cosign sign-blob --key="/var/lib/jenkins/cosign/keys/artf.key" --output="/var/lib/jenkins/cosign/sign-files/artf" -y ./wolf/target/wolf-0.0.1-SNAPSHOT.jar'
                }
            }
        }
    }
  }

  //post { 
        //success { 
            //cleanWs()
        //}
    //}
}