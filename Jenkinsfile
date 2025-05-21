pipeline {
  agent any

  stages {
      stage('Build Artifact - MAVEN') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
        stage('UNIT TEST - JUNIT and JACOCO') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'     // JUnit
                jacoco execPattern: 'target/jacoco.exec'  // Jacoco 
              }
            }
        }    
    }
}