pipeline {
  agent none
  stages {
    stage('worker - build') {
      agent {
        docker {
          image 'maven:3.6.1'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/*'
      }
      steps {
        echo 'Compiling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker - test') {
      agent {
        docker {
          image 'maven:3.6.1'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        changeset '**/worker/*'
      }
      steps {
        echo 'Running Unit Test on worker app.'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker - package') {
      agent {
        docker {
          image 'maven:3.6.1'
          args '-v $HOME/.m2:/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/*'
      }
      steps {
        echo 'Packaging worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker - docker-package') {
      agent any
      when {
        changeset '**/worker/*'
      }
      steps {
        echo 'Packaging worker app in Docker container'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'hub.docker.com') {
            def workerImage = docker.build("yadkov/worker:${env.BUILD_ID}", "./worker/")
            workerImage.push()
            workerImage.push("latest")
          }
        }

      }
    }

    stage('result - build') {
      agent {
        docker {
          image 'node:8.9.0'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'step build'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result - test') {
      agent {
        docker {
          image 'node:8.9.0'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Test on result app.'
        dir(path: 'result') {
          sh 'npm test'
        }

      }
    }

    stage('result - docker-package') {
      agent any
      when {
        changeset '**/result/*'
      }
      steps {
        echo 'Packaging result app in Docker container'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'hub.docker.com') {
            def resultImage = docker.build("yadkov/result:${env.BUILD_ID}", "./result/")
            resultImage.push()
            resultImage.push("latest")
          }
        }

      }
    }

    stage('vote - build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'step build'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote - test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Test on vote app.'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote integration'){
      agent any
      when{
        changeset "**/vote/**"
        branch 'master'
      }
      steps{
        echo 'Running Integration Tests on vote app'
        dir('vote'){
          sh 'sh integration_test.sh'
        }
      }
    }

    stage('vote - docker-package') {
      agent any
      when {
        branch 'master'
        changeset '**/vote/*'
      }
      steps {
        echo 'Packaging vote app in Docker container'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'hub.docker.com') {
            def voteImage = docker.build("yadkov/vote:${env.BUILD_ID}", "./vote/")
            voteImage.push()
            voteImage.push("latest")
          }
        }

      }
    }

    stage('Deploy to dev') {
      agent any
      when {
        branch 'master'
      }
      steps {
        echo 'Execute docker-compose'
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Building multibranch pipeline for worker is completed..'
    }

    failure {
      slackSend(channel: 'instavote-cd', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

    success {
      slackSend(channel: 'instavote-cd', message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

  }
}
