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

    stage('Docker Build Backend') {
    steps {
        sh '''
        cd demo
        docker build -t shouqmajed/app-backend:latest .
        '''
    }
}

    stage('Docker Push Backend') {
  steps {
    withCredentials([usernamePassword(
      credentialsId: 'DockerHub',
      usernameVariable: 'DOCKERHUB_USER',
      passwordVariable: 'DOCKERHUB_PASS'
    )]) {
      sh '''
        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
        docker push shouqmajed/app-backend:latest
      '''
    }
  }
}

  }
}
