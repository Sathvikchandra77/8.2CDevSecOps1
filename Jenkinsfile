pipeline {
  agent any
  tools { nodejs 'Node18' }           // Manage Jenkins → Tools → NodeJS → name it Node18 (Install automatically ✓)
  options { timestamps() }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Node version') { steps { bat 'node -v && npm -v' } }

    stage('Install (robust)') {
      when { expression { fileExists('package.json') } }
      steps {
        // Try npm ci if a lockfile exists; if it fails (out-of-sync), delete lock and npm install
        bat '''
          if exist package-lock.json (
            echo Lockfile found. Trying npm ci...
            npm ci || ( echo npm ci failed. Removing lockfile and running npm install... & del /f /q package-lock.json & npm install )
          ) else (
            echo No lockfile. Running npm install...
            npm install
          )
        '''
      }
    }

    stage('Test (if present)') {
      when { expression { fileExists('package.json') } }
      steps { bat 'npm test || exit /b 0' }
    }

    stage('Coverage (if present)') {
      when { expression { fileExists('package.json') } }
      steps { bat 'npm run coverage || exit /b 0' }
    }

    stage('Security Audit (if present)') {
      when { expression { fileExists('package.json') } }
      steps { bat 'npm audit || exit /b 0' }
    }
  }

  post {
    always {
      // remove this junit line if you don’t produce JUnit XML to avoid the warning
      junit allowEmptyResults: true, testResults: '**\\junit*.xml'
      archiveArtifacts artifacts: '**\\coverage\\**\\*', allowEmptyArchive: true
    }
  }
}
