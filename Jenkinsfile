pipeline {
  agent any

  environment {
    JAVA_HOME = "/usr/lib/jvm/java-21-openjdk-amd64"
    PATH = "${JAVA_HOME}/bin:${env.PATH}"
  }

  stages {

    stage('Backend - Build') {
      steps {
        dir('demo') {
          sh '''
            echo "JAVA_HOME=$JAVA_HOME"
            which java
            java -version
            which javac
            javac -version
            mvn -v

            mvn -U clean package -DskipTests=true \
              -Dmaven.compiler.fork=true \
              -Dmaven.compiler.executable=$JAVA_HOME/bin/javac
          '''
        }
      }
    }

    stage('Backend - Test') {
      steps {
        dir('demo') {
          sh '''
            mvn test \
              -Dmaven.compiler.fork=true \
              -Dmaven.compiler.executable=$JAVA_HOME/bin/javac
          '''
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
