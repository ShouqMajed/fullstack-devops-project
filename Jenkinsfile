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

  }
}
