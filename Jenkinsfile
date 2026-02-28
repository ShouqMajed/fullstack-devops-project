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
               nexusUrl: '15.188.28.246:8081',
               nexusVersion: 'nexus3',
               protocol: 'http',
               repository: 'fullstack-backend',
               version: "0.0.1-${BUILD_NUMBER}"
           }
         }
      }

      stage('Upload-Frontend-to-Nexus') {
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
          }

stage('Sonarqube') {
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

}

stage('Sonarqube-Frontend') {
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
