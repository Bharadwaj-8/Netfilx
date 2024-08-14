// pipeline{
//     agent any
//     tools{
//         jdk 'jdk'
//         nodejs 'nodejs'
//     }
//     environment {
//         SCANNER_HOME=tool 'sonar-server'
//     }
//     stages {
//         stage('Workspace Cleaning'){
//             steps{
//                 cleanWs()
//             }
//         }
//         stage('Checkout from Git'){
//             steps{
//                 git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/Netflix-Clone-K8S-End-to-End-Project.git'
//             }
//         }
//         stage("Sonarqube Analysis"){
//             steps{
//                 withSonarQubeEnv('sonar-server') {
//                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
//                     -Dsonar.projectKey=Netflix \
//                     '''
//                 }
//             }
//         }
//         stage("Quality Gate"){
//            steps {
//                 script {
//                     waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
//                 }
//             } 
//         }
//         stage('Install Dependencies') {
//             steps {
//                 sh "npm install"
//             }
//         }
//         stage('OWASP DP SCAN') {
//             steps {
//                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check'
//                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
//             }
//         }
//         stage('TRIVY FS SCAN') {
//             steps {
//                 sh "trivy fs . > trivyfs.txt"
//             }
//         }
//         stage("Docker Image Build"){
//             steps{
//                 script{
//                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
//                        sh "docker system prune -f"
//                        sh "docker container prune -f"
//                        sh "docker build --build-arg TMDB_V3_API_KEY=8b174e589e2f03f9fd8123907bd7800c -t netflix ."
//                     }
//                 }
//             }
//         }
//         stage("Docker Image Pushing"){
//             steps{
//                 script{
//                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
//                        sh "docker tag netflix avian19/netflix:latest "
//                        sh "docker push avian19/netflix:latest "
//                     }
//                 }
//             }
//         }
//         stage("TRIVY Image Scan"){
//             steps{
//                 sh "trivy image avian19/netflix:latest > trivyimage.txt" 
//             }
//         }
//         stage('Deploy to Kubernetes'){
//             steps{
//                 script{
//                     dir('Kubernetes') {
//                         withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
//                                 sh 'kubectl apply -f deployment.yml'
//                                 sh 'kubectl apply -f service.yml'
//                                 sh 'kubectl get svc'
//                                 sh 'kubectl get all'
//                         }   
//                     }
//                 }
//             }
//         }
//     }
//     post {
//      always {
//         emailext attachLog: true,
//             subject: "'${currentBuild.result}'",
//             body: "Project: ${env.JOB_NAME}<br/>" +
//                 "Build Number: ${env.BUILD_NUMBER}<br/>" +
//                 "URL: ${env.BUILD_URL}<br/>",
//             to: 'aman07pathak@gmail.com',
//             attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
//         }
//     }
// }

pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' // Replace with your AWS region
        ECR_REPO_NAME = 'netflix'
        ECR_URI = "public.ecr.aws/n8f6h3y2/netflix"
        KUBE_CONFIG_PATH = '/home/ubuntu/.kube/config' // Update with the correct path
    }

    stages {
        stage('Login to ECR') {
            steps {
                script {
                    sh "aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/n8f6h3y2"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_URI}:latest")
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    docker.image("${ECR_URI}:latest").push()
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig-credentials', kubeconfig: KUBE_CONFIG_PATH]) {
                        sh '''
                        kubectl set image deployment/netflix-app netflix-app=${ECR_URI}:latest
                        kubectl apply -f deployment.yml
                        kubectl apply -f service.yml
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}

