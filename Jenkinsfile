pipeline {
  agent any

  environment {
    UAT_ENV   = 'uat'
    UAT_APIM  = 'https://172.22.50.136:9443'
    INSECURE  = '-k'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Configure apictl env (UAT)') {
      steps {
        sh '''
          set -e
          apictl add env ${UAT_ENV} --apim ${UAT_APIM} 2>/dev/null || true
          apictl get envs | grep -q "^${UAT_ENV}$" || (echo "UAT env not added" && exit 1)
        '''
      }
    }

    stage('Login to UAT') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'apim-uat-admin',
          usernameVariable: 'APIM_USER',
          passwordVariable: 'APIM_PASS'
        )]) {
          sh '''
            set -e
            echo "$APIM_PASS" | apictl login ${UAT_ENV} -u "$APIM_USER" --password-stdin ${INSECURE}
          '''
        }
      }
    }

    stage('Import APIs to UAT') {
      steps {
        sh '''
          set -e

          # Import every API project folder that contains api.yaml
          for d in */; do
            if [ -f "${d}api.yaml" ]; then
              echo "Importing API project: $d"
              apictl import api -e ${UAT_ENV} -f "$d" ${INSECURE} --update --verbose
            fi
          done
        '''
      }
    }

    stage('Verify') {
      steps {
        sh '''
          set -e
          apictl get apis -e ${UAT_ENV} ${INSECURE}
        '''
      }
    }
  }
}

