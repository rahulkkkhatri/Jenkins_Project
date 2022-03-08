pipeline {
   agent any
   environment {     
    DOCKERHUB_CREDENTIALS= credentials('DockerHub')     
   }
   options {
        office365ConnectorWebhooks([
            [name: "Office 365", url: "https://lntinfotech.webhook.office.com/webhookb2/6b347fc6-ad12-4379-a6f3-584ca6974c75@02aa9fc1-18bc-4798-a020-e01c854dd434/JenkinsCI/86179d88b7484e40927eb8781bdbd0fa/8ff106e7-829e-4ab0-be75-636bb50c366e", notifyBackToNormal: true, notifyFailure: true, notifyRepeatedFailure: true, notifySuccess: true, notifyAborted: true]
        ])
   }
   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }
      stage('Docker Build') {
         steps {
            sh(script: 'docker images -a')
            sh(script: """
               cd azure-vote/
               docker images -a
               docker build -t docker0rahul/jenkins_project:$BUILD_NUMBER .
               docker images -a
               cd ..
            
            """)
         }
      }
      stage('Start Test App') {
         steps {
            sh(script: """
              docker-compose up -d
              chown jenkins:jenkins ./scripts/test_container.sh
              chmod 777 ./scripts/test_container.sh
              ./scripts/test_container.sh
            """)

         }
         post {
            success {
               echo "App started Successfully"
            }
            failure {
               echo "App failed to start"
            }
         }
      }
      stage('Run Test') {
         steps {
            sh(script: """
              chown jenkins:jenkins ./tests/test_sample.py
              chmod 777 ./tests/test_sample.py
              pytest ./tests/test_sample.py
            """)
         }
      }
      stage('Stop Tests') {
         steps {
            sh(script: """
              docker-compose down
            """)
         }
      }
      stage('Push Image to DockerHub'){
         steps {
            echo "Workspace is $WORKSPACE"
            sh "docker login -u docker0rahul -p $DOCKERHUB_CREDENTIALS"
            sh "docker push docker0rahul/jenkins_project:$BUILD_NUMBER"
            sh "docker logout"
         }
      }
      
      stage('Container Scanning') {
         parallel {
            stage('Run Anchore') {
               steps {
                  echo "Run Anchore"
                  /*sh "echo 'docker0rahul/jenkins_project:$BUILD_NUMBER' > anchore_images"
                  anchore bailOnFail: false, bailOnPluginFail: false, name: 'anchore_images'*/
               }
            }
            stage('Run Trivy') {
               steps {
                  sh "trivy image docker0rahul/jenkins_project:$BUILD_NUMBER > trivy_report"
               }
            }
         }
      }
   }
}
     