pipeline {
  agent any
  options { timestamps() }
  triggers { pollSCM('H/5 * * * *') }  // or use a GitHub webhook

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/<your_user>/8.2CDevSecOps.git'
      }
    }
    stage('Node Version') {
      steps {
        bat 'node -v && npm -v'   // sanity check PATH
      }
    }
    stage('Install') {
      steps { bat 'npm ci || npm install' }
    }
    stage('Test') {
      steps { bat 'npm test || exit /b 0' }     // don’t fail build for demo
    }
    stage('Coverage') {
      steps { bat 'npm run coverage || exit /b 0' }
    }
    stage('Security Audit') {
      steps { bat 'npm audit || exit /b 0' }
    }
  }
  post {
    always {
      junit allowEmptyResults: true, testResults: '**\\junit*.xml'
      archiveArtifacts artifacts: '**\\coverage\\**\\*', allowEmptyArchive: true
    }
  }
}
