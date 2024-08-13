pipeline {
  agent any

  // Assuming you have configured tools 'jdk' and 'nodejs' in Jenkins
  // tools {
  //   jdk 'jdk'
  //   nodejs 'nodejs'
  // }

  environment {
    // ... (any environment variables needed for deployment)
  }

  stages {
    stage('Workspace Cleaning') {
      steps {
        cleanWs()
      }
    }

    stage('Checkout from Git') {
      steps {
        git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/Netflix-Clone-K8S-End-to-End-Project.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        sh "npm install"
      }
    }

    stage("Docker Image Build") {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
            sh "docker system prune -f"
            sh "docker container prune -f"
            sh "docker build --build-arg TMDB_V3_API_KEY=8b174e589e2f03f9fd8123907bd7800c -t netflix ."
          }
        }
      }
    }

    stage("Docker Image Pushing") {
      steps {
        script {
          withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
            sh "docker tag netflix avian19/netflix:latest"
            sh "docker push avian19/netflix:latest"
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          dir('Kubernetes') {
            withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
              sh 'kubectl apply -f deployment.yml'
              sh 'kubectl apply -f service.yml'
              sh 'kubectl get svc'
              sh 'kubectl get all'
            }
          }
        }
      }
    }
  }

  post {
    always {
      emailext attachLog: true,
        subject: "'${currentBuild.result}'",
        body: "Project: ${env.JOB_NAME}<br/>" +
              "Build Number: ${env.BUILD_NUMBER}<br/>" +
              "URL: ${env.BUILD_URL}<br/>",
        to: 'aman07pathak@gmail.com',
        attachmentsPattern: '' // Remove attachments since scanning is removed
    }
  }
}
