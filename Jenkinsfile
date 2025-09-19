pipeline {
  agent any
  options { timestamps() }
  triggers { pollSCM('H/5 * * * *') } // or set a GitHub webhook later
  stages {
    stage('Checkout') {
      steps { checkout scm }  // pulls this repo
    }
    stage('Node version') {  // remove if not a Node project
      steps { bat 'node -v && npm -v' }
    }
    stage('Install') {
      steps { bat 'npm ci || npm install' }
    }
    stage('Test') {
      steps { bat 'npm test || exit /b 0' }   // don’t fail build during demo
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

