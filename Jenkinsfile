pipeline {
  agent any
  tools {
        maven 'Maven3.9'
      }
  stages {

    stage("Cloning Git Repo") {
      steps {
        git branch: 'main', credentialsId: 'personal-GitHub-Creds', url: 'https://github.com/hemant-1905/kubernetes-files-for-argo-CD.git'
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
          sh 'docker build -t hemaant07/devops-integration:$BUILD_NUMBER .'
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
          sh 'docker push hemaant07/devops-integration:$BUILD_NUMBER'
        }
      }
    }

  stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "kubernetes-files-for-argo-CD"
            GIT_USER_NAME = "hemant-1905"
        }
        steps {
            withCredentials([string(credentialsId: 'github-connector', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "hemaant07@gmail.com"
                    git config user.name "Hemant Sharma"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i '' -E 's/image: hemaant07/devops-integration: .*/${BUILD_NUMBER}/' kubernetes-files/deploymentservice.yaml
                    git add  kubernetes-files/deploymentservice.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }

  }
}
