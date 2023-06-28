pipeline {
  agent any
  tools {
        maven 'Maven3.9'
      }
  stages {

    stage("Cloning Git Repo") {
      steps {
        git branch: 'main', credentialsId: 'personal-GitHub-Creds', url: 'https://github.com/hemant-1905/java-app-image-on-Dockerhub.git'
      }
    }

    stage('SonarQube Analysis') {
      steps{
    withSonarQubeEnv(credentialsId: 'sonar-user-credentials', installationName: 'Sonarqube-jenkins') {
      sh 'mvn sonar:sonar'
    }
    }
  }

    stage("building maven build") {     
      steps {
        sh 'mvn clean install package'
      }
    }
    stage("Building Docker image") {
      steps {
        script {
          sh 'docker build -t hemaant07/devops-integration:latest .'
        }
      }
    }

stage('Trivy Scan') {
steps{
       sh 'trivy image --severity CRITICAL --format json --output trivy-scan_results.json hemaant07/devops-integration:latest'
} 
        }


    stage("Deploy to DockerHub") {
      steps {
        script {
          withCredentials([string(credentialsId: 'docker_pwd', variable: 'docker_password')]) {
            sh 'docker login -u hemaant07 -p ${docker_password}'
          }
          sh 'docker push hemaant07/devops-integration:latest'
        }
      }
    }

    stage("Delete Existing K8 Objects") {
      steps {
        sh 'kubectl delete deployment spring-boot-k8s-deployment --ignore-not-found=true'
        sh 'kubectl delete service springboot-k8ssvc --ignore-not-found=true'
      }
   }

    stage("Deploy app to Kubernetes Cluster") {
      steps {
        sh 'kubectl apply -f deploymentservice.yaml'
      }
    }

  }
}