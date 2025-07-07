pipeline {
  agent { label 'agent'}

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Unit Tests') {
      when { branch 'dev' }
      steps {
        dir('javaapp-pipeline') {
          sh 'mvn clean test'
        }
      }
    }

    stage('Sonar Analysis') {
      when { branch 'dev' }
      steps {
        dir('javaapp-pipeline') {
          withSonarQubeEnv('sonar') {
            sh """
              mvn clean verify sonar:sonar \
                -Dsonar.projectKey=java-app \
                -Dsonar.branch.name=${env.BRANCH_NAME}
            """
          }
        }
      }
    }

    stage('Package') {
      when { branch 'main' }
      steps {
        dir('javaapp-pipeline') {
          sh 'mvn clean package'
        }
      }
    }

    stage('Deploy') {
      when { branch 'main' }
      steps {
        dir('javaapp-pipeline/target') {
          sh '''
            if pgrep -f "java -jar java-sample-*.jar" > /dev/null; then
              pkill -f "java -jar java-sample-*.jar"
              echo "App was running and has been killed."
            else
              echo "App is not running."
            fi
            JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-*.jar > app.log 2>&1 &
          '''
        }
      }
    }
  }
}
