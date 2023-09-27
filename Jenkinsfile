pipeline {
  agent any 

  environment {
    registry = "573314280082.dkr.ecr.ap-south-1.amazonaws.com/pixalive"
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
            docker build -t pixalive:v1 .
          """  
        }
      }
    }

    stage('Image Push to ECR') {
      steps {
        script {
          sh """
            aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 573314280082.dkr.ecr.ap-south-1.amazonaws.com
            docker tag pixalive:v1 573314280082.dkr.ecr.ap-south-1.amazonaws.com/pixalive:v1
            docker push 573314280082.dkr.ecr.ap-south-1.amazonaws.com/pixalive:v1
          """
        }
      }
    }
  }
}
