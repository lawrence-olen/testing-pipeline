pipeline {
  agent any

  environment {
    REPO_URL = 'https://github.com/lawrence-olen/testing-pipeline.git'
    GIT_CREDENTIALS = 'master-testing'
    GIT_BRANCH = 'production'
    DIR_BUILD = '/home/crocox/retail-store'
    IMAGE_TAG_UI = 'crocoxolen/retail-store-sample-ui:production'
    IMAGE_TAG_CART = 'crocoxolen/retail-store-sample-cart:production'
    IMAGE_TAG_CATALOG = 'crocoxolen/retail-store-sample-catalog:production'
    IMAGE_TAG_ORDERS = 'crocoxolen/retail-store-sample-orders:production'
    IMAGE_TAG_CHECKOUT = 'crocoxolen/retail-store-sample-checkout:production'
    IMAGE_TAG_ASSETS = 'crocoxolen/retail-store-sample-assets:production'
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
          uiChanged = sh(script: "git diff --name-only ${previousCommit} HEAD | grep '^src/ui/' || true", returnStdout: true).trim()
          cartChanged = sh(script: "git diff --name-only ${previousCommit} HEAD | grep '^src/cart/' || true", returnStdout: true).trim()
          catalogChanged = sh(script: "git diff --name-only ${previousCommit} HEAD | grep '^src/catalog/' || true", returnStdout: true).trim()
          ordersChanged = sh(script: "git diff --name-only ${previousCommit} HEAD | grep '^src/orders/' || true", returnStdout: true).trim()
          checkoutChanged = sh(script: "git diff --name-only ${previousCommit} HEAD | grep '^src/checkout/' || true", returnStdout: true).trim()
          assetsChanged = sh(script: "git diff --name-only ${previousCommit} HEAD | grep '^src/assets/' || true", returnStdout: true).trim()

          // Flag untuk menentukan apakah ada perubahan di setiap service
          hasUiChanges = uiChanged ? true : false
          hasCartChanges = cartChanged ? true : false
          hasCatalogChanges = catalogChanged ? true : false
          hasOrdersChanges = ordersChanged ? true : false
          hasCheckoutChanges = checkoutChanged ? true : false
          hasAssetsChanges = assetsChanged ? true : false

          if (hasUiChanges) {
            echo "Changes detected in src/ui: ${hasUiChanges}"
          } else {
            echo "No Changes detected in src/ui"
          }

          if (hasCartChanges) {
            echo "Changes detected in src/cart: ${hasCartChanges}"
          } else {
            echo "No Changes detected in src/cart"
          }

          if (hasCatalogChanges) {
            echo "Changes detected in src/catalog: ${hasCatalogChanges}"
          } else {
            echo "No Changes detected in src/catalog"
          }

          if (hasOrdersChanges) {
            echo "Changes detected in src/orders: ${hasOrdersChanges}"
          } else {
            echo "No Changes detected in src/orders"
          }

          if (hasCheckoutChanges) {
            echo "Changes detected in src/checkout: ${hasCheckoutChanges}"
          } else {
            echo "No Changes detected in src/checkout"
          }

          if (hasAssetsChanges) {
            echo "Changes detected in src/assets: ${hasAssetsChanges}"
          } else {
            echo "No Changes detected in src/assets"
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
                  docker buildx build --no-cache -t ${IMAGE_TAG_UI} src/ui
                '
                """
              }
            }
          }
        }

        stage ("Cart Service") {
          when {
            expression { return hasCartChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_BUILD} &&
                  docker buildx build --no-cache -t ${IMAGE_TAG_CART} src/cart
                '
                """
              }
            }
          }
        }

        stage ("Catalog Service") {
          when {
            expression { return hasCatalogChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_BUILD} &&
                  docker buildx build --no-cache -t ${IMAGE_TAG_CATALOG} src/catalog
                '
                """
              }
            }
          }
        }

        stage ("Orders Service") {
          when {
            expression { return hasOrdersChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_BUILD} &&
                  docker buildx build --no-cache -t ${IMAGE_TAG_ORDERS} src/orders
                '
                """
              }
            }
          }
        }

        stage ("Checkout Service") {
          when {
            expression { return hasCheckoutChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_BUILD} &&
                  docker buildx build --no-cache -t ${IMAGE_TAG_CHECKOUT} src/checkout
                '
                """
              }
            }
          }
        }

        stage ("Assets Service") {
          when {
            expression { return hasAssetsChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_BUILD} &&
                  docker buildx build --no-cache -t ${IMAGE_TAG_ASSETS} src/assets
                '
                """
              }
            }
          }
        }
      }
    }

    stage ("Push Docker Image") {
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
                  docker push ${IMAGE_TAG_UI}
                  docker system prune -a -f
                '
                """
              } 
            }
          }
        }

        stage ("Cart Service") {
          when {
            expression { return hasCartChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  docker push ${IMAGE_TAG_CART}
                  docker system prune -a -f
                '
                """
              }
            }
          }
        }

        stage ("Catalog Service") {
          when {
            expression { return hasCatalogChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  docker push ${IMAGE_TAG_CATALOG}
                  docker system prune -a -f
                '
                """
              }
            }
          }
        }

        stage ("Orders Service") {
          when {
            expression { return hasOrdersChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  docker push ${IMAGE_TAG_ORDERS}
                  docker system prune -a -f
                '
                """
              }
            }
          }
        }

        stage ("Checkout Service") {
          when {
            expression { return hasCheckoutChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  docker push ${IMAGE_TAG_CHECKOUT}
                  docker system prune -a -f
                '
                """
              }
            }
          }
        }

        stage ("Assets Service") {
          when {
            expression { return hasAssetsChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  docker push ${IMAGE_TAG_ASSETS}
                  docker system prune -a -f
                '
                """
              }
            }
          }
        }
      }
    }

    stage ("Deployment Service on Retail Store") {
      parallel {
        stage ("Deploy UI Service") {
          when {
            expression { return hasUiChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_DEPLOY}/ui &&
                  helm upgrade -f values.yaml -n default ui .
                '
                """
              }
            }
          }
        }

        stage ("Deploy Cart Service") {
          when {
            expression { return hasCartChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_DEPLOY}/carts &&
                  helm upgrade -f values.yaml -n default carts .
                '
                """
              }
            }
          }
        }

        stage ("Deploy Catalog Service") {
          when {
            expression { return hasCatalogChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_DEPLOY}/catalog &&
                  helm upgrade -f values.yaml -n default catalog .
                '
                """
              }
            }
          }
        }

        stage ("Deploy Orders Service") {
          when {
            expression { return hasOrdersChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_DEPLOY}/orders &&
                  helm upgrade -f values.yaml -n default orders .
                '
                """
              }
            }
          }
        }

        stage ("Deploy Checkout Service") {
          when {
            expression { return hasCheckoutChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_DEPLOY}/checkout &&
                  helm upgrade -f values.yaml -n default checkout .
                '
                """
              }
            }
          }
        }

        stage ("Deploy Assets Service") {
          when {
            expression { return hasAssetsChanges }
          }

          steps {
            script {
              sshagent(credentials: [env.GIT_CREDENTIALS]) {
                sh """
                ssh -o StrictHostKeyChecking=no ${MASTER_TESTING_SERVER} '
                  cd ${DIR_DEPLOY}/assets &&
                  helm upgrade -f values.yaml -n default assets .
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
