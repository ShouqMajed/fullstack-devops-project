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

    stage('Upload-to-Nexus') {
      steps {
        dir('demo') {
            nexusArtifactUploader artifacts: [
               [
                 artifactId: 'demo',
                 classifier: '',
                 file: 'target/demo-0.0.1-SNAPSHOT.jar',
                 type: 'jar'
               ]
            ],
                credentialsId: 'nexus',
                groupId: 'com.example',
               nexusUrl: '35.181.173.92:8081',
               nexusVersion: 'nexus3',
               protocol: 'http',
               repository: 'fullstack-backend',
               version: "0.0.1-${BUILD_NUMBER}"
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
                ansible-playbook ansible/playbook-docker.yml -e "workspace=$WORKSPACE tag=$BUILD_NUMBER"
              '''
             }
           }
        }

  }
}
