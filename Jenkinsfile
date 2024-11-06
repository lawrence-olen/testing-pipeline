pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/lawrence-olen/testing-pipeline.git'
    GIT_CREDENTIALS = 'master-testing'
    GIT_BRANCH = 'production'
  }

  stages {
    stage ("Checkout and Pull Repository") {
      script {
        git url: "${REPO_URL}", credentialsId: "${GIT_CREDENTIALS}", branch: "${GIT_BRANCH}"
      }
    }
  }
}
