pipeline {
  agent any

  stages {
      stage('Build Artifact - MAVEN') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
        stage('UNIT TEST') {
            steps {
              sh "mvn test"
            }
        }    
    }
}