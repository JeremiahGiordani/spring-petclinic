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

    stage('Deploy to Production VM') {
      steps {
        sshagent(['petclinic-vm-key']) {
          sh '''
            JAR=$(ls target/spring-petclinic-*.jar | head -1)
            # Package the controller's JDK to ship to the VM (VM has no internet)
            tar czf jdk17.tgz -C /opt/java openjdk
            ANSIBLE_HOST_KEY_CHECKING=False \
            ansible-playbook -i ansible/inventory.ini ansible/deploy.yml \
              -e "jar_src=${WORKSPACE}/${JAR}" \
              -e "jdk_src=${WORKSPACE}/jdk17.tgz"
          '''
        }
      }
    }
  }
}
