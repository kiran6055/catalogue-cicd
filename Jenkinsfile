pipeline {
  agent any 
  parameters {
    string(name: 'VERSION', description: 'Enter the APP VERSION')
    string(name: 'APP_NAME', description: 'Enter the APPLICATION NAME')
  }

  environment {
    REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/catalogue"
    REGION = "ap-south-1"
    AWS_ACCOUNT_ID = "573314280082"
  }

  stages {
    stage('gitpull') {
      steps {
        script {
          cleanWs() // Corrected step name
          git branch: 'main', url: "https://github.com/kiran6055/catalogue-cicd"
        }
      }
    }

    stage('Docker Build') {
      steps {
        script {
          sh """
            docker build -t catalogue:${params.VERSION} .
          """  
        }
      }
    }

    stage('Image Push to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_AUTH']]) {
          script {
            sh """
              aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.ap-south-1.amazonaws.com
              docker tag catalogue:${params.VERSION} ${REGISTRY}:${params.VERSION}
              docker push ${REGISTRY}:${params.VERSION}
            """
          }
        }
      }
    }

    stage('Deploying into Kubernetes') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_AUTH']]) {
          script {
            sh """
              aws eks update-kubeconfig --region ${REGION} --name spot-cluster
              cd helm

              helm upgrade ${params.APP_NAME} . --set deployment.imageVersion=${params.VERSION}
            """
          }
        }
      }
    }
  }
}


// #              helm install ${params.APP_NAME} . --set deployment.imageVersion=${params.VERSION}
