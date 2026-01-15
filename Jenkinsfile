pipeline {
  agent any

  environment {
    APIM_UAT = "uat"
    APIM_UAT_URL = "https://172.22.50.136:9443"
  }

  stages {

    stage('Init apictl env') {
      steps {
        sh '''
          apictl add env ${APIM_UAT} --apim ${APIM_UAT_URL} || true
        '''
      }
    }

    stage('Login to UAT') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'uat-apim-creds',
          usernameVariable: 'UAT_USER',
          passwordVariable: 'UAT_PASS'
        )]) {
          sh '''
            echo "$UAT_PASS" | apictl login ${APIM_UAT} -u $UAT_USER --password-stdin -k
          '''
        }
      }
    }

    stage('Import API to UAT') {
      steps {
        sh '''
          apictl import api -f ColTrainScheduleCommunityAPI-1.0.0 \
            -e ${APIM_UAT} \
            --update \
            --preserve-provider \
            -k
        '''
      }
    }

    stage('Deploy to Gateway') {
      steps {
        sh '''
          apictl deploy api -n ColTrainScheduleCommunityAPI \
            -v 1.0.0 \
            -e ${APIM_UAT} \
            --gateway-environment Production \
            -k
        '''
      }
    }
  }
}

