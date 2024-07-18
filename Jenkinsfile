pipeline {
  agent any
  environment {
      repo_name="${MicroServices}"
      branch_name="${Branch}"
      release_type="${Release}"
      COSIGN_PASSWORD=credentials('11aad1b0-d788-4813-ab4a-81fd4bbb7487')
      AWS_ACCESS_KEY = credentials('7a5f353a-4c21-491f-a4a6-cb4738f5b0a9') 
      AWS_SECRET_KEY = credentials('590a8e9b-8854-431e-817c-08faef36799d')
      AWS_REGION = credentials('efcaf984-bf4c-4f34-a84c-5f0438bfcdba')
      AWS_ACCOUNT_ID = credentials('c620055a-b75b-40b2-a390-d780f977faa8')
  }

  stages {
    stage('Git Clone') {
        steps {
          script {
            if ("${repo_name}" == "core") {
              dir('acs-connector') {
                git(url: 'https://git.assistanz.com/stackbill/acs-connector.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              dir('wolf') {
                git(url: 'https://git.assistanz.com/stackbill/wolf.git', credentialsId: 'ebf87b99-0a18-4b01-a994-55c51a857e7b', branch: "${branch_name}")
              }
              sh 'mv ./wolf/Dockerfile Dockerfile && mv ./wolf/.dockerignore .dockerignore'
              sh 'ls -al'
            }
            if ("${repo_name}" == "billing") {
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
            if ("${repo_name}" == "core") {
              sh '''
                sudo update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/bin/java
                java -version && mvn -version
                cd ./acs-connector/ && mvn clean install -U -DskipTests=true && cd ..
                cp ./acs-connector/target/connectors-1.0.0-SNAPSHOT.jar ./wolf
                cd ./wolf && mvn clean package -DskipTests=true && cd ..
                mkdir artifacts && cp ./wolf/target/*.jar artifacts/
                sudo update-alternatives --set java /usr/lib/jvm/java-17-openjdk-amd64/bin/java

              '''
            }
            if ("${repo_name}" == "billing") {
              sh '''
                sudo update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/bin/java
                java -version && mvn -version
                cd ./acs-connector/ && mvn clean install -U -DskipTests=true && cd ..
                cp ./acs-connector/target/connectors-1.0.0-SNAPSHOT.jar ./odolf
                cd ./odolf && mvn clean package -DskipTests=true && cd ..
                mkdir artifacts && cp ./odolf/target/*.jar artifacts/
                sudo update-alternatives --set java /usr/lib/jvm/java-17-openjdk-amd64/bin/java
              '''
            }
          }
        }
    }

    stage('Signing Artifact') {
        steps {
            script {
                if(fileExists('/var/lib/jenkins/cosign/keys/artf.key')) {
                    sh 'COSIGN_PASSWORD=$COSIGN_PASSWORD cosign sign-blob --key="/var/lib/jenkins/cosign/keys/artf.key" --output="/var/lib/jenkins/cosign/sign-files/artf" -y ./artifacts/*.jar'
                }
                else {
                    sh 'COSIGN_PASSWORD=$COSIGN_PASSWORD cosign generate-key-pair --output-key-prefix="/var/lib/jenkins/cosign/keys/artf"'
                    sh 'COSIGN_PASSWORD=$COSIGN_PASSWORD cosign sign-blob --key="/var/lib/jenkins/cosign/keys/artf.key" --output="/var/lib/jenkins/cosign/sign-files/artf" -y ./artifacts/*.jar'
                }
            }
        }
    }

    stage('S3 Upload') {
        steps {  
            /** Upload to S3**/
          script {
            withAWS(region:'ap-south-1', credentials:'67917c46-3b5e-4cd1-91e9-9d4c3bb7f7de') {
               //def identity=awsIdentity();
               s3Upload(bucket:"stackbill-artifacts", workingDir: 'artifacts', includePathPattern:'*.jar');
            }
          }
        }
    }

    stage('Image Build') {
      steps {
        sh '''
          podman rmi --all
          cp ./artifacts/*.jar .
          if [ "$repo_name" = "core" ]; then
            podman build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi .
          elif [ "$repo_name" = "billing" ]; then
            podman build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-billing .
          fi
        '''
      }
    }

    stage('ECR Push') {
      steps {
        sh '''
          aws configure set aws_access_key_id ${AWS_ACCESS_KEY}
          aws configure set aws_secret_access_key ${AWS_SECRET_KEY}
          aws configure set region ${AWS_REGION}
          aws ecr get-login-password --region ${AWS_REGION} | podman login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
          if [ "$repo_name" = "core" ]; then
            if [ "$branch_name" = "stable" ]; then
              image_tag=$(aws ecr describe-images --repository-name stackbill-coreapi --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[0]' | grep -v "alpha" | grep -v "beta" | awk 'NR==2{print $1}' | tr -d '"' | tr -d ',' | tr -d 'v')
              if [ "$release_type" = "Major" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d"." -f1`
                j=0
                k=0
                i=$(expr $i + 1)
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag
              elif [ "$release_type" = "Minor" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d"." -f1`
                j=`echo $image_tag | awk "{print $1}" | cut -d"." -f2`
                k=0
                if [ "$j" -gt 1000 ]; then
                  j=0
                  i=$(expr $i + 1)
                else
                  j=$(expr $j + 1)
                fi
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag
              elif [ "$release_type" = "Patch" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d "." -f1`
                j=`echo $image_tag | awk "{print $1}" | cut -d "." -f2`
                k=`echo $image_tag | awk "{print $1}" | cut -d "." -f3`
                if [ "$k" -gt 20 ]; then
                  exit;
                else
                  k=$(expr $k + 1)
                fi
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag
              fi
            fi

            if [ "$branch_name" = "development" ]; then
              image_tag=$(aws ecr describe-images --repository-name stackbill-coreapi --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[0]' | grep -v "alpha" | grep -v "beta" | awk 'NR==2{print $1}' | tr -d '"' | tr -d ',' | tr -d 'v')
              if [ "$release_type" = "Major" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d"." -f1`
                j=0
                k=0
                i=$(expr $i + 1)
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-alpha
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-alpha
              elif [ "$release_type" = "Minor" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d"." -f1`
                j=`echo $image_tag | awk "{print $1}" | cut -d"." -f2`
                k=0
                if [ "$j" -gt 1000 ]; then
                  j=0
                  i=$(expr $i + 1)
                else
                  j=$(expr $j + 1)
                fi
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-alpha
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-alpha
              elif [ "$release_type" = "Patch" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d "." -f1`
                j=`echo $image_tag | awk "{print $1}" | cut -d "." -f2`
                k=`echo $image_tag | awk "{print $1}" | cut -d "." -f3`
                if [ "$k" -gt 20 ]; then
                  exit;
                else
                  k=$(expr $k + 1)
                fi
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-alpha
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-alpha
              fi
            fi

            if [ "$branch_name" = "pre-stable" ]; then
              image_tag=$(aws ecr describe-images --repository-name stackbill-coreapi --query 'sort_by(imageDetails,& imagePushedAt)[*].imageTags[0]' | grep -v "alpha" | grep -v "beta" | awk 'NR==2{print $1}' | tr -d '"' | tr -d ',' | tr -d 'v')
              if [ "$release_type" = "Major" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d"." -f1`
                j=0
                k=0
                i=$(expr $i + 1)
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-beta
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-beta
              elif [ "$release_type" = "Minor" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d"." -f1`
                j=`echo $image_tag | awk "{print $1}" | cut -d"." -f2`
                k=0
                if [ "$j" -gt 1000 ]; then
                  j=0
                  i=$(expr $i + 1)
                else
                  j=$(expr $j + 1)
                fi
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-beta
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-beta
              elif [ "$release_type" = "Patch" ]; then
                i=`echo $image_tag | awk "{print $1}" | cut -d "." -f1`
                j=`echo $image_tag | awk "{print $1}" | cut -d "." -f2`
                k=`echo $image_tag | awk "{print $1}" | cut -d "." -f3`
                if [ "$k" -gt 20 ]; then
                  exit;
                else
                  k=$(expr $k + 1)
                fi
                new_tag=v$i.$j.$k
                podman tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-beta
                podman push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/stackbill-coreapi:$new_tag-beta
              fi
            fi
          fi
        '''
      }
    }
  }

  post { 
        success { 
            cleanWs()
        }
    }
}