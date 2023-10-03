def call () {
  try {

    pipeline {
      agent any 
      parameters {
        string(name: 'VERSION', description: 'Enter the APP VERSION')
        string(name: 'APP_NAME', description: 'Enter the APPLICATION NAME')
      }

      environment {
        REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${APP_NAME}"
        REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "358308582535"
      }

      stages {
        stage('gitpull') {
          steps {
            script {
              cleanWs() // Corrected step name
              git credentialsId: '	pixalive-github', branch: 'main', url: 'https://github.com/Tripalive/pixalive-story-server.git'

            }
          }
        }

        stage('Docker Build') {
          steps {
            script {
              sh """
                docker build -t ${APP_NAME}:${VERSION} .
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
                  docker tag ${APP_NAME}:${VERSION} ${REGISTRY}:${VERSION}
                  docker push ${REGISTRY}:${VERSION}
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
                  aws eks update-kubeconfig --region ${REGION} --name kubernetes-cluster
                  cd helm
                  helm install  ${APP_NAME}  . --set deployment.imageVersion=${VERSION}

                """
              }
            }
          }
        }
      }
    }
  } catch (Exception e) {
    common.email("Failed")
  }
}

// #              helm install ${params.APP_NAME} . --set deployment.imageVersion=${params.VERSION}
//  helm upgrade $APP_NAME} . --set deployment.imageVersion=${params.VERSION}


in common.groovy file in vars section we need to place it

def email(email_note) {
  mail bcc: '', body: "Job Failed - ${JOB_BASE_NAME}\nJenkins URL - ${JOB_URL}", cc: '', from: 'kiran@pixalive.me', replyTo: '', subject: "Jenkins Job Failed - ${JOB_BASE_NAME}", to: 'kiran@pixalive.me, kabeer@pixalive.me'
}

// in the above \n means new line below