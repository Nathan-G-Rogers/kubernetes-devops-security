pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }   
      stage('Unit Tests') {
            steps {
              sh "mvn test"
            }
    }
    stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            }
        } 
    }
    stage('Build and Push Image') {
        steps {
            script {
                docker.withRegistry('https://gcr.io', 'gcr:k8s-dev-sec-ops') {
                  def customImage = docker.build("k8s-dev-sec-ops/numeric-app:${env.GIT_COMMIT}")
                  customImage.push()
                }
            }
        }
    }
    stage('K8s Deployment - Dev') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#k8s-dev-sec-ops/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
    }
  }
}