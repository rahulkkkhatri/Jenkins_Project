pipeline {
   agent any
   environment {
       webhook_url = "https://lntinfotech.webhook.office.com/webhookb2/6b347fc6-ad12-4379-a6f3-584ca6974c75@02aa9fc1-18bc-4798-a020-e01c854dd434/JenkinsCI/86179d88b7484e40927eb8781bdbd0fa/8ff106e7-829e-4ab0-be75-636bb50c366e"                               //can be used in whole pipeline
       result = ""
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
               docker build -t jenkins-pipeline .
               docker images -a
               cd ..
            
            """)
         }
         post {
            success {
                office365ConnectorSend webhookUrl: "${webhook_url}",
                result: "${result} ${STAGE_NAME}: Image Build Success!!",
                status: 'Success'            
            }
            failure {
                office365ConnectorSend webhookUrl: "${webhook_url}",
                result: "${result} ${STAGE_NAME}: Image Build Failed!!",
                status: 'Failure'            
            }
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
                office365ConnectorSend webhookUrl: "${webhook_url}",
                result: "${result} '\n'${STAGE_NAME}: App started successfully!!",
                status: 'Success'            
            }
            failure {
                office365ConnectorSend webhookUrl: "${webhook_url}",
                result: "${result} '\n'${STAGE_NAME}: App start Failed",
                status: 'Failure'            
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
      
   }
   post{
      success {
         office365ConnectorSend webhookUrl: "${webhook_url}",
                message: "${result}",
                status: 'Success'
      }
      failure {
         office365ConnectorSend webhookUrl: "${webhook_url}",
                result: "${result}",
                status: 'Failed'
      }
   }
}
     