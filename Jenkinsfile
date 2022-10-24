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
    stage('Sonar Qube - SAST') {
          steps {
            withSonarQubeEnv('SonarQube')
            sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric_application -Dsonar.host.url=http://34.67.170.30:9000 -Dsonar.login=sqp_e295bc5976aa25cdf455c00b71d6fcbe1b98f913"
          }
    }
    stage('Sonar Qube - quality gate') {
      steps {
      timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
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