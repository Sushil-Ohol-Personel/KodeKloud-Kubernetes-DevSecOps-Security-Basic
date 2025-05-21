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
         stage('K8S Deployment - DEV') {
            steps {
                 withKubeConfig([credentialsId: 'kubeconfig']) {
                 //sh "sed -i "s#replace#${imageName}#g" k8s_deployment_service.yaml"
                 sh "sed -i "s#replace#sushil1234/sushil-kodekloud-devsecops-repo:${GIT_COMMIT}#g" k8s_deployment_service.yaml"
                 sh "kubectl apply -f k8s_deployment_service.yaml"
         }
      }
    }
    //   stage('K8S Deployment - DEV') {
    //     steps {
    //       parallel(
    //         "Deployment": {
    //             withKubeConfig([credentialsId: 'kubeconfig']) {
    //             sh "bash k8s-deployment.sh"
    //         }
    //       },
    //         "Rollout Status": {
    //             withKubeConfig([credentialsId: 'kubeconfig']) {
    //             sh "bash k8s-deployment-rollout-status.sh"
    //         }
    //       }
    //     )
    //   }
    // }
  }
}