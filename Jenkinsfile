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
            post {
                     always {
                      junit 'target/surefire-reports/*.xml'
                      jacoco execPattern: 'target/jacoco.exec'
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
  }
}