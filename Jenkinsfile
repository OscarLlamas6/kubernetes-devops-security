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
    }
}
