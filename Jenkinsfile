pipeline {
  agent any

      environment {
    EMAIL_TO = "shouq.majed.a@gmail.com"    
  }

  stages {

    stage('Backend - Build') {
      steps {
        dir('demo') {
          sh 'mvn clean package -DskipTests=true'
        }
      }
                  post {
                success { notifyStage('Backend - Build', 'SUCCESS') }
                  failure { notifyStage('Backend - Build', 'FAILURE') }
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
                  post {
                success { notifyStage('Backend - Test', 'SUCCESS') }
                  failure { notifyStage('Backend - Test', 'FAILURE') }
             }
    }

    stage('Frontend - Install') {
      steps {
        dir('frontend') {
          sh 'npm ci'
        }
      }
                  post {
                success { notifyStage('Frontend - Install', 'SUCCESS') }
                  failure { notifyStage('Frontend - Install', 'FAILURE') }
             }
    }

    stage('Frontend - Build') {
      steps {
        dir('frontend') {
          sh 'npm run build'
        }
      }
                  post {
                success { notifyStage('Frontend - Build', 'SUCCESS') }
                  failure { notifyStage('Frontend - Build', 'FAILURE') }
             }
    }

    stage('Frontend - Test') {
      steps {
        dir('frontend') {
          sh 'xvfb-run -a npx ng test --watch=false --browsers=ChromeHeadless'
        }
      }
                  post {
                success { notifyStage('Frontend - Test', 'SUCCESS') }
                  failure { notifyStage('Frontend - Test', 'FAILURE') }
             }
    }

stage('SonarQube Analysis') {
  parallel {
    stage('Backend - Sonarqube') {
      steps {
        dir('demo') {
          withSonarQubeEnv('sonarqube') {
            sh """
              mvn -DskipTests clean verify \
              org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
              -Dsonar.projectKey=fullstack-backend
            """
          }
        }

        timeout(time: 5, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
      post {
        success { notifyStage('Backend - Sonarqube', 'SUCCESS') }
        failure { notifyStage('Backend - Sonarqube', 'FAILURE') }
      }
    }

    stage('Frontend - Sonarqube') {
      steps {
        dir('frontend') {
          withSonarQubeEnv('sonarqube') {
            script {
              def scannerHome = tool 'sonar-scanner'
              sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=fullstack-frontend \
                -Dsonar.sources=src
              """
            }
          }
        }

        timeout(time: 5, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
      post {
        success { notifyStage('Frontend - Sonarqube', 'SUCCESS') }
        failure { notifyStage('Frontend - Sonarqube', 'FAILURE') }
      }
    }
  }
}

stage('Nexus Upload') {
  parallel {
    stage('Backend-to-Nexus') {
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
          nexusUrl: '15.188.28.246:8081',
          nexusVersion: 'nexus3',
          protocol: 'http',
          repository: 'fullstack-backend',
          version: "0.0.1-${BUILD_NUMBER}"
        }
      }
      post {
        success { notifyStage('Backend-to-Nexus', 'SUCCESS') }
        failure { notifyStage('Backend-to-Nexus', 'FAILURE') }
      }
    }

    stage('Frontend-to-Nexus') {
      steps {
        dir('frontend') {
          sh "tar -czf frontend-0.0.1-${BUILD_NUMBER}.tgz dist"
          nexusArtifactUploader artifacts: [
            [
              artifactId: 'frontend',
              classifier: '',
              file: "frontend-0.0.1-${BUILD_NUMBER}.tgz",
              type: 'tgz'
            ]
          ],
          credentialsId: 'nexus',
          groupId: 'com.example',
          nexusUrl: '15.188.28.246:8081',
          nexusVersion: 'nexus3',
          protocol: 'http',
          repository: 'fullstack-frontend',
          version: "0.0.1-${BUILD_NUMBER}"
        }
      }
      post {
        success { notifyStage('Frontend-to-Nexus', 'SUCCESS') }
        failure { notifyStage('Frontend-to-Nexus', 'FAILURE') }
      }
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
                   post {
                success { notifyStage('Docker Build & Push', 'SUCCESS') }
                  failure { notifyStage('Docker Build & Push', 'FAILURE') }
             }
        }

    stage('Deploy to Kubernetes (Ansible)') {
  steps {
    dir('ansible') {
      sh 'ansible-playbook -i inventory.ini playbook-k8s.yaml'
    }
  }
                  post {
                success { notifyStage('Deploy to Kubernetes', 'SUCCESS') }
                  failure { notifyStage('Deploy to Kubernetes', 'FAILURE') }
             }
}

  }
               post {
                success { notifyStage('PIPELINE', 'SUCCESS') }
                  failure { notifyStage('PIPELINE', 'FAILURE') }
             }
}
// ===== Function to send email using Email Extension Plugin =====
def notifyStage(String stageName, String status) {
  emailext(
    to: env.EMAIL_TO,
    subject: "[Jenkins] ${env.JOB_NAME} #${env.BUILD_NUMBER} | ${stageName} | ${status}",
    mimeType: 'text/html',
    body: """
      <h3>Pipeline Notification</h3>
      <p><b>Job:</b> ${env.JOB_NAME}</p>
      <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
      <p><b>Stage:</b> ${stageName}</p>
      <p><b>Status:</b> ${status}</p>
      <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
    """
  )
}

