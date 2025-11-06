
pipeline {
  agent any

  environment {
    APP_ID = "fhxR-9lCOYnAoXHQo9HeU" 
  }

  parameters {
    string(name: 'DEPLOY_URL_CRED_ID', defaultValue: 'DEPLOY_URL', description: 'Credentials ID for the deployment URL (Secret Text)')
    string(name: 'DEPLOY_KEY_CRED_ID', defaultValue: 'DEPLOY_KEY', description: 'Credentials ID for the deployment API key (Secret Text)')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Env setup') {
      steps {
        sh 'python3 -m venv .venv'
      }
    }

    stage('Install requirements') {
      steps {
        sh '.venv/bin/pip install -r requirements.txt'
      }
    }

    stage('Migrations') {
      steps {
        sh '.venv/bin/python manage.py makemigrations'
        sh '.venv/bin/python manage.py migrate'
      }
    }

    stage('Unit Tests') {
      steps {
        sh '.venv/bin/python manage.py test'
      }
    }

    stage('Start Server') {
      steps {
        sh '.venv/bin/python manage.py runserver 0.0.0.0:8000 &'
        sh 'sleep 5' // wait for server to boot
      }
    }

    stage('API Test') {
      steps {
        sh '.venv/bin/python check.py'
      }
    }
  }  
  post {
    success {
      echo "✅ Tests passed, triggering deployment API..."
      withCredentials([
        string(credentialsId: params.DEPLOY_URL_CRED_ID, variable: 'DEPLOY_URL'),
        string(credentialsId: params.DEPLOY_KEY_CRED_ID, variable: 'DEPLOY_KEY')
      ]) {
        sh '''
          json_payload=$(printf '{"applicationId":"%s"}' "$APP_ID")
          curl -fS -X POST \
            "$DEPLOY_URL" \
            -H 'accept: application/json' \
            -H 'Content-Type: application/json' \
            -H "x-api-key: $DEPLOY_KEY" \
            --data-binary "$json_payload" \
            -w "\nHTTP %{http_code}\n"
        '''
      }
      // Send success email notification
      mail to: 'xaioene@gmail.com',
           subject: "Jenkins Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) has succeeded.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Deployment API was triggered successfully.

Regards,
Jenkins
"""
    }

    failure {
      echo "❌ Pipeline failed, sending error email..."
      mail to: 'xaioene@gmail.com',
           subject: "Jenkins Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           body: """Hello,

The Jenkins pipeline for ${env.JOB_NAME} (build #${env.BUILD_NUMBER}) has failed.

* Branch: ${env.BRANCH_NAME}
* Commit: ${env.GIT_COMMIT}
* Build URL: ${env.BUILD_URL}

Please check the console output for details.

Regards,
Jenkins
"""
    }
  }
}

