pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/lawrence-olen/testing-pipeline.git'
    GIT_CREDENTIALS = 'master-testing'
    GIT_BRANCH = 'production'
  }

  stages {
    stage ("Checkout and Pull Repository") {
      steps {
        git url: "${REPO_URL}", credentialsId: "${GIT_CREDENTIALS}", branch: "${GIT_BRANCH}"
      }
    }

    stage ("Detect Changes") {
      steps {
        script {  
          // Mengambil commit sebelumnya
          def previousCommit = sh(script: 'git rev-parse HEAD^', returnStdout: true).trim()
          // Periksa perubahan di setiap service
          uiChanged = sh(script: "git diff --name-only ${previousCommit} HEAD grep '^src/ui/' || true", returnStdout: true).trim()
          cartChanged = sh(script: "git diff --name-only ${previousCommit} HEAD grep '^src/cart/' || true", returnStdout: true).trim()

          // Flag untuk menentukan apakah ada perubahan di setiap service
          hasUiChanges = uiChanged ? true : false
          hasCartChanges = cartChanged ? true : false

          if (hasUiChanges) {
            echo "Changes detected in src/ui: ${hasUiChanges}"
          }

          if (hasCartChanges) {
            echo "Changes detected in src/cart: ${hasCartChanges}"
          }
        }
      }
    }

    stage ("Build Docker Images") {
      parallel {
        stage ("UI Service") {
          when {
            expression { return hasUiChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_BUILD} &&
                  docker buildx build --no-cache -t ${IMAGE_TAG} src/ui
                '
                """
              }
            }
          }
        }
      }
    }
  }
}
