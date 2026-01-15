pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    // Target APIM env name in apictl
    UAT_ENV  = 'uat'
    UAT_APIM = 'https://172.22.50.136:9443'

    // apictl flags
    INSECURE = '-k'

    // Make apictl state local to this workspace (avoids /var/lib/jenkins cross-job clashes)
    HOME = "${WORKSPACE}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Verify apictl installed') {
      steps {
        sh '''
          set -e
          command -v apictl >/dev/null 2>&1 || { echo "apictl not found on agent"; exit 1; }
          apictl version
        '''
      }
    }

    stage('Configure apictl env (UAT)') {
      steps {
        sh '''
          set -e

          # Add env if missing (safe to run every time)
          apictl add env ${UAT_ENV} --apim ${UAT_APIM} 2>/dev/null || true

          echo "Configured environments:"
          apictl get envs || true

          # Basic sanity check
          apictl get envs | grep -qw "${UAT_ENV}" || (echo "UAT env '${UAT_ENV}' not present in apictl config" && exit 1)
        '''
      }
    }

    stage('Login to UAT') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'apim-uat-admin',
          usernameVariable: 'APIM_USER',
          passwordVariable: 'APIM_PASS'
        )]) {
          sh '''
            set -e
            echo "$APIM_PASS" | apictl login ${UAT_ENV} -u "$APIM_USER" --password-stdin ${INSECURE} --verbose
          '''
        }
      }
    }

    stage('Import APIs to UAT') {
      steps {
        sh '''
          set -e

          found=0

          # Import every API project folder that contains api.yaml
          for d in */ ; do
            if [ -f "${d}api.yaml" ]; then
              found=1
              echo "===================================================="
              echo "Importing API project: ${d}"
              echo "===================================================="
              apictl import api -e ${UAT_ENV} -f "${d%/}" ${INSECURE} --update --preserve-provider --verbose
            fi
          done

          if [ "$found" -eq 0 ]; then
            echo "No API projects found (expected folders with api.yaml in repo root)."
            exit 1
          fi
        '''
      }
    }

    stage('Verify import') {
      steps {
        sh '''
          set -e
          echo "APIs currently in UAT:"
          apictl get apis -e ${UAT_ENV} ${INSECURE} --verbose
        '''
      }
    }
  }

  post {
    always {
      sh '''
        echo "Workspace: $WORKSPACE"
        echo "apictl config (if present):"
        ls -lah "$HOME/.wso2apictl" 2>/dev/null || true
        ls -lah "$HOME/.wso2apictl.local" 2>/dev/null || true
      '''
    }
  }
}

