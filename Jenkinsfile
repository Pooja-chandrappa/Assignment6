pipeline {
  agent { label 'agent' }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Unit Tests') {
      when { branch 'dev' }
      steps { sh 'mvn clean test' }
    }

    stage('Package') {
      when { anyOf { branch 'main'; branch 'master' } }
      steps { sh 'mvn clean package' }
    }

    stage('Deploy') {
      when { anyOf { branch 'main'; branch 'master' } }
      steps {
        sh '''
          if pgrep -f "java -jar target/java-sample-*.jar" > /dev/null; then
            pkill -f "java -jar target/java-sample-*.jar"
            echo "App was running and has been killed."
          else
            echo "App is not running."
          fi
          JENKINS_NODE_COOKIE=dontKillMe nohup java -jar target/java-sample-*.jar > app.log 2>&1 &
        '''
      }
    }
  }

  post {
    always { echo "Pipeline finished on branch ${env.BRANCH_NAME}" }
    failure { echo "Pipeline failed on branch ${env.BRANCH_NAME}" }
  }
}
