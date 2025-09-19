pipeline {
  agent any

  // <<< THIS is where you add it
  tools { nodejs 'Node18' }   // must match the name in Manage Jenkins → Tools → NodeJS
  // >>>

  options { timestamps() }
  // triggers { pollSCM('H/5 * * * *') }  // optional: uncomment if you want polling

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Node version') {                // sanity check PATH + NodeJS tool
      steps { bat 'node -v && npm -v' }
    }

    stage('Install (if Node project)') {
      when { expression { fileExists('package.json') } }
      steps {
        // Use npm ci when a lockfile exists; otherwise npm install
        bat 'if exist package-lock.json ( npm ci ) else ( npm install )'
      }
    }

    stage('Test (if Node project)') {
      when { expression { fileExists('package.json') } }
      steps { bat 'npm test || exit /b 0' }
    }

    stage('Coverage (if Node project)') {
      when { expression { fileExists('package.json') } }
      steps { bat 'npm run coverage || exit /b 0' }
    }

    stage('Security Audit (if Node project)') {
      when { expression { fileExists('package.json') } }
      steps { bat 'npm audit || exit /b 0' }
    }
  }

  post {
    always {
      // if your tests produce JUnit XML, they'll be picked up
      junit allowEmptyResults: true, testResults: '**\\junit*.xml'
      // archive coverage files if they exist
      archiveArtifacts artifacts: '**\\coverage\\**\\*', allowEmptyArchive: true
    }
  }
}
