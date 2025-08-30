pipeline {
  agent any

  stages {
      stage('Build Artifacts') {
          steps {
            sh "mvn clean package -DskipTests=true"
            archive 'target/*.jar'
          }
      }

      stage('Unit Tests - JUnit & Jacoco') {
          steps {
            sh "mvn test"
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }
      }

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "dockerhub-creds", url: ""]) {
            sh 'printenv'
            sh 'sudo docker build -t oscarllamas6/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push oscarllamas6/numeric-app:""$GIT_COMMIT""'
          }
        }
      }

    }
}
