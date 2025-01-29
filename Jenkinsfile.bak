pipeline {
  agent {any}
  options {
    buildDiscarder logRotate(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')
    disableConcurrentBuild()
  }
  stages('Hello') {
    stage('Hello') {
      steps {
        echo "Hello"
      }
    }
  }
}
