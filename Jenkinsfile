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
         stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                sh 'printenv'
                sh 'docker build -t sushil1234/sushil-kodekloud-devsecops-repo:""$GIT_COMMIT"" .'
                sh 'docker push sushil1234/sushil-kodekloud-devsecops-repo:""$GIT_COMMIT""'
                }
            }
    } 
  }
}