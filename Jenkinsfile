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
// To Test
        // stage('SonarQube - SAST') {
        //     steps {
        //       withSonarQubeEnv('sonarqube') {
        //         sh "mvn sonar:sonar \
        //             -Dsonar.projectKey=numeric-application \
        //             -Dsonar.projectName='numeric-application' \
        //             -Dsonar.host.url=http://3.140.157.179:9000 "
        //             // -Dsonar.token=sqp_f4b4ce18b985a44d5b8761b36bbea4b7fb508303"
        //         // sh "mvn sonar:sonar \
        //         //         -Dsonar.projectKey=numeric-application \
        //         //         -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000"
        //       }
        //       timeout(time: 2, unit: 'MINUTES') {
        //         script {
        //           waitForQualityGate abortPipeline: true
        //         }
        //       }
        //     }
        //   }

        stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
                sh 'printenv'
                sh 'docker build -t sushil1234/sushil-kodekloud-devsecops-repo:""$GIT_COMMIT"" .'
                sh 'docker push sushil1234/sushil-kodekloud-devsecops-repo:""$GIT_COMMIT""'
                }
            }
    } 
        stage('K8S Deployment - DEV') {
            steps {
                 withKubeConfig([credentialsId: 'kubeconfig']) {
                 sh "sed -i 's#replace#sushil1234/sushil-kodekloud-devsecops-repo:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                 sh "kubectl apply -f k8s_deployment_service.yaml"
         }
      }
    }
  }
}