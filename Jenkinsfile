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
              python3 ./tests/test_sample.py
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
}
     