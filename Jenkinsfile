pipeline {
  agent any

  environment {
    APIM_ENV = "uat"
    API_DIR  = "apis/ColTrainScheduleCommunityAPI/1.0.0"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('apictl init env') {
      steps {
        sh '''
          apictl add env uat --apim https://172.22.50.136:9443 || true
        '''
      }
    }

    stage('Login to UAT') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'apim-uat-creds',
                         usernameVariable: 'UAT_USER',
                         passwordVariable: 'UAT_PASS')]) {
          sh '''
            apictl login uat -u "$UAT_USER" -p "$UAT_PASS" -k
          '''
        }
      }
    }

    stage('Import API Project to UAT') {
      steps {
        sh '''
          # --update is important for idempotency (import again updates existing API)
          apictl import api -e uat -f "$API_DIR" -k --update
        '''
      }
    }

    stage('Create & Deploy Revision (Gateway)') {
      steps {
        sh '''
          # Depending on your apictl version/APIM config, you typically:
          # 1) create api revision
          # 2) deploy it to a gateway environment

          # If your apictl supports it, do something like:
          # apictl create api revision -e uat -n ColTrainScheduleCommunityAPI -v 1.0.0 -k
          # apictl deploy api revision -e uat -n ColTrainScheduleCommunityAPI -v 1.0.0 -k --target-gateway-env "Default"
          #
          # If your setup auto-deploys on import (some tutorial packs do), you can skip this stage.
          #
          apictl get apis -e uat -k
        '''
      }
    }
  }
}

