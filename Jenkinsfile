pipeline {
   agent any

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
      }
      stage('Start Test App') {
         steps {
            sh(script: """
              docker compose up
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
              pytest ./tests/test_sample.py
            """)
         }
      }
      stage('Stop Tests') {
         steps {
            sh(script: """
              docker compose down
            """)
         }
      }
      
   }
}
     