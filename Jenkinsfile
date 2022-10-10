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
      stage('Docker Build and Push') {
            steps {
              withDockerRegistry([credentialsId: "k8s-dev-sec-op", url: "gcr.io"]) {
                sh "printenv"
                sh 'docker build -t gcr.io/k8s-dev-sec-ops/numeric-app:""$GIT_COMMIT"" .'
                sh 'docker push gcr.io/k8s-dev-sec-ops/numeric-app:""$GIT_COMMIT""'
         } 
      }
    }
  }
}