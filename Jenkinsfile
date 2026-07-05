pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'chmod +x mvnw'
      }
    }

    stage('Build & Test') {
      steps {
        // DB integration tests need Docker (unavailable in the controller), so
        // exclude them; all unit tests (controllers, services, validators) run.
        sh "./mvnw -B clean verify -Dtest='!PostgresIntegrationTests,!MySqlIntegrationTests' -Dsurefire.failIfNoSpecifiedTests=false"
      }
      post {
        always {
          junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "./mvnw -B sonar:sonar -Dsonar.projectKey=spring-petclinic -Dsonar.projectName='spring-petclinic'"
        }
      }
    }
  }
}
