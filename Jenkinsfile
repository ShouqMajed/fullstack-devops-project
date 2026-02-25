pipeline {
  agent any

  stages {

    stage('Backend - Build') {
      steps {
        dir('demo') {
          sh 'mvn clean package -DskipTests=true'
        }
      }
    }

    stage('Backend - Test') {
      environment {
        SPRING_PROFILES_ACTIVE = 'test-no-db'
      }
      steps {
        dir('demo') {
          sh 'mvn test'
        }
      }
    }

    stage('Frontend - Install') {
      steps {
        dir('frontend') {
          sh 'npm ci'
        }
      }
    }

    stage('Frontend - Build') {
      steps {
        dir('frontend') {
          sh 'npm run build'
        }
      }
    }

    stage('Frontend - Test') {
      steps {
        dir('frontend') {
          sh 'xvfb-run -a npx ng test --watch=false --browsers=ChromeHeadless'
        }
      }
    }

     stage('Docker Build & Push (Ansible)') {
       steps {
          withCredentials([usernamePassword(
          credentialsId: 'DockerHub',
           usernameVariable: 'DOCKERHUB_USER',
           passwordVariable: 'DOCKERHUB_PASS'
         )]) {
            sh '''
                ansible-playbook ansible/playbook-docker.yml -e "workspace=$WORKSPACE"
              '''
             }
           }
        }

  }
}
