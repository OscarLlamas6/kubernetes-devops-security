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

      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }

      stage('SonarQube Analysis') {
        steps {
          withSonarQubeEnv(installationName: 'oscarplayground-sonarqube') { 
            sh "mvn clean verify sonar:sonar -Dsonar.projectKey=oscarplayground -Dsonar.projectName='oscarplayground'"
          }
        }
      }

      stage('Vulnerability Scan') {
        steps {
            withCredentials([string(credentialsId: 'depcheck-oscarplayground', variable: 'NVD_API_KEY')]) {
                sh """
                    mvn org.owasp:dependency-check-maven:8.4.0:check \
                    -Dnvd.api.key=\$NVD_API_KEY \
                    -Dformat=XML
                """
            }
        }
        post {
            always {
                dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
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

      stage('K8S Deployment - DEV') {
        steps {          
            withKubeConfig([credentialsId: 'kubectl-creds']) {
                sh "sed -i 's#replace#oscarllamas6/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"             
            }
        }
      }

    }
}
