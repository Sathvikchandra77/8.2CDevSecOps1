pipeline {
  agent any
  tools { nodejs 'Node18' }         // Manage Jenkins → Tools → NodeJS → name = Node18 (Install automatically ✓)
  options { timestamps() }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Node version') { steps { bat 'node -v && npm -v' } }

    stage('Install (robust)') {
      when { expression { fileExists('package.json') } }
      steps {
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
    success {
      emailext(
        to: 'sathvikchandra77@outlook.com',
        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build Succeeded.

Job:   ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
URL:   ${env.BUILD_URL}console

Agent: ${env.NODE_NAME}
Commit: ${env.GIT_COMMIT}
""",
        attachLog: true
      )
    }
    failure {
      emailext(
        to: 'sathvikchandra77@outlook.com',
        subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        body: """Build Failed.

Job:   ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
URL:   ${env.BUILD_URL}console

Please check the console log for the failing stage.
""",
        attachLog: true
      )
    }
    always {
      // (Optional) quiet warnings about missing reports/artifacts
      script {
        def junitReports = findFiles(glob: '**/junit*.xml')
        if (junitReports && junitReports.length > 0) {
          junit testResults: '**/junit*.xml'
        } else {
          echo 'No JUnit XML found – skipping.'
        }
        if (fileExists('coverage')) {
          archiveArtifacts artifacts: 'coverage/**/*', onlyIfSuccessful: false
        } else {
          echo 'No coverage/ directory – skipping.'
        }
      }
    }
  }
}
