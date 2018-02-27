pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        checkout scm
        sh '''
          sudo docker build -t $IMAGE_NAME:$BUILD_ID .
        '''
      }
    }
    stage('Testing') {
      steps {
        echo 'TODO: add tests'
      }
    }
    stage('Create Image & Release') {
      when {
        expression {
          env.BRANCH_NAME == 'master'
        }
        
      }
      steps {
        sh 'echo $(aws ecr get-login --region us-east-2 --registry-ids 410602862282) > file.txt'
        sh 'sudo $( sed "s/-e none//g" file.txt)'
        sh 'sudo  docker tag $IMAGE_NAME:${BUILD_ID} 410602862282.dkr.ecr.us-east-2.amazonaws.com/demo-jenkins-pipeline:${BUILD_ID}'
        sh 'sudo docker push 410602862282.dkr.ecr.us-east-2.amazonaws.com/demo-jenkins-pipeline:${BUILD_ID}'
      }
    }
    stage('Staging Deployment') {
      when {
        expression {
          env.BRANCH_NAME == 'master'
        }
        
      }
      environment {
        RELEASE_NAME = 'demo-staging'
        SERVER_HOST = 'staging.hm.zone'
      }
      steps {
        sh '''
          helm upgrade --install --namespace staging $RELEASE_NAME ./helm/kubedemo --set image.tag=$BUILD_ID,ingress.host=$SERVER_HOST
        '''
      }
    }
    stage('Deploy to Production?') {
      when {
        expression {
          env.BRANCH_NAME == 'master'
        }
        
      }
      steps {
        milestone 1
        input 'Deploy to Production?'
        milestone 2
      }
    }
    stage('Production Deployment') {
      when {
        expression {
          env.BRANCH_NAME == 'master'
        }
        
      }
      environment {
        RELEASE_NAME = 'demo-production'
        SERVER_HOST = 'production.hm.zone'
      }
      steps {
        sh '''
          helm upgrade --install --namespace production $RELEASE_NAME ./helm/kubedemo --set image.tag=$BUILD_ID,ingress.host=$SERVER_HOST
        '''
      }
    }
  }
  environment {
    IMAGE_NAME = 'demohelm'
  }
}
