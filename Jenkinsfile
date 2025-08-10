pipeline {
  agent any

  environment {
    ARM_CLIENT_ID         = credentials('azure-client-id')
    ARM_CLIENT_SECRET     = credentials('azure-client-secret')
    ARM_TENANT_ID         = credentials('azure-tenant-id')
    ARM_SUBSCRIPTION_ID   = credentials('azure-subscription-id')
  }
  
  stages {
    stage('Checkout Code') {
      steps {
        git credentialsId: 'github-credentials',
            url: 'https://github.com/annkur-sharma/AzureCode04-JenkinsPipeline',
            branch: 'main'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('MySonarQubeServer') {
          bat 'sonar-scanner'
        }
      }
    }
    
    stage('Terraform Init') {
      steps {
        dir('INFRA01') {
          bat 'terraform init -backend-config="backend-config-infra01.hcl"'
        }
      }
    }

    stage('Terraform Plan') {
      steps {
         dir('INFRA01') {
           bat 'terraform plan -var "main_provider_subscription_id=%ARM_SUBSCRIPTION_ID%" -var "user_prefix=eagle987" -out=tfplan'
        }
      }
    }

    stage('Approval') {
      steps {
        script {
          def userInput = input(
            id: 'Proceed',
            message: 'Do you approve to proceed to deployment?',
            parameters: [
              choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Select environment')
            ]
            )
            echo "Approval received for environment: ${userInput}"
          }
        }
      }

    stage('Terraform Apply') {
      steps {
         dir('INFRA01') {
           bat 'terraform apply tfplan'
        }
      }
    }   
  }
}

