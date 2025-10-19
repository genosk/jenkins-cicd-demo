pipeline {
  agent any

  environment {
    BUILD_CONFIG = 'Release'
    PUBLISH_DIR = 'published'
    ZIP_FILE = 'published.zip'
  }

  stages {

    stage('Checkout Code') {
      steps {
        echo "🔹 Cloning code from GitHub..."
        git branch: 'main', url: 'https://github.com/genosk/jenkins-cicd-demo.git'
      }
    }

    stage('Restore Dependencies') {
      steps {
        echo "🔹 Restoring .NET dependencies..."
        bat "dotnet restore"
      }
    }

    stage('Build') {
      steps {
        echo "🔹 Building project..."
        bat "dotnet build --configuration %BUILD_CONFIG%"
      }
    }

    stage('Test') {
      steps {
        echo "🔹 Running tests (if any)..."
        bat "dotnet test --configuration %BUILD_CONFIG% || exit 0"
      }
    }

    stage('Publish Artifact') {
      steps {
        echo "🔹 Publishing build output..."
        bat "dotnet publish -c %BUILD_CONFIG% -o ${PUBLISH_DIR}"

        echo "🔹 Creating ZIP archive..."
        powershell """
          if (Test-Path '${ZIP_FILE}') { Remove-Item '${ZIP_FILE}' -Force }
          Compress-Archive -Path '${PUBLISH_DIR}\\*' -DestinationPath '${ZIP_FILE}'
        """

        archiveArtifacts artifacts: "${ZIP_FILE}", fingerprint: true
      }
    }

    stage('Deploy Locally') {
      steps {
        echo "🔹 Deploying locally to IIS folder..."
        powershell """
          $targetPath = 'C:\\inetpub\\wwwroot\\jenkins-cicd-demo'
          if (Test-Path $targetPath) {
            Remove-Item $targetPath -Recurse -Force
          }
          New-Item -ItemType Directory -Path $targetPath -Force | Out-Null
          Expand-Archive -Path '${ZIP_FILE}' -DestinationPath $targetPath
        """
      }
    }
  }

  post {
    success {
      echo "✅ Build and local deployment successful!"
      echo "App is available at http://localhost/jenkins-cicd-demo"
    }
    failure {
      echo "❌ Build or deployment failed. Check Jenkins logs for details."
    }
  }
}
