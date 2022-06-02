pipeline {
  agent {
    docker {
      image 'jenkins/inbound-agent'
    }

  }
  stages {
    stage('git clone') {
      steps {
        git(branch: 'master', url: 'https://github.com/gongjixiaobai/simple-java-maven-app.git')
      }
    }

  }
}