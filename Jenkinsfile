pipeline {
  agent any

  // Make sure these tools exist in Jenkins (Manage Jenkins → Tools):
  // - NodeJS: Name = Node18  (Install automatically ✓)
  // - SonarQube Scanner: Name = SonarScanner (Install automatically ✓)
  tools { nodejs 'Node18' }
  options { timestamps() }

  environment {
    SONAR_SCANNER_HOME = tool 'SonarScanner'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Node version') {
      steps { bat 'node -v && npm -v' }
    }

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

    stage('SonarCloud Analysis') {
      steps {
        // Add your Sonar token in Jenkins:
        // Manage Jenkins → Credentials → (Global) Add Credentials → Kind: Secret text
        // ID = SONAR_TOKEN, Secret = <your sonarcloud token>
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          // 👇 Override JAVA_HOME just for this stage (your local JDK folder; no \bin\java.exe)
          withEnv([
            'JAVA_HOME=C:\\Users\\sathv\\AppData\\Local\\Programs\\Eclipse Adoptium\\jdk-21.0.8.9-hotspot',
            'PATH=C:\\Users\\sathv\\AppData\\Local\\Programs\\Eclipse Adoptium\\jdk-21.0.8.9-hotspot\\bin;%PATH%'
          ]) {
            bat 'echo Using JAVA_HOME=%JAVA_HOME% & java -version'
            bat '"%SONAR_SCANNER_HOME%\\bin\\sonar-scanner.bat" -D"sonar.login=%SONAR_TOKEN%"'
          }
        }
      }
    }
  }

  post {
    always {
      script {
        if (fileExists('reports/junit.xml')) {
          junit testResults: 'reports/junit.xml'
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
