pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'webapijenkinspratham22025'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/chouhanpratham/WebApiJenkins.git'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet restore'
                sh 'dotnet build --configuration Release'
                sh 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Package') {
            steps {
                sh 'tar -cvf publish.tar ./publish'  // Using tar instead of zip
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    script {
                        env.AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
                        env.AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
                        env.AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
                    }
                    sh "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
                    
                    // Deploy using tar file
                    sh """
                    az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME \
                    --src-path ./publish.tar --type zip --restart
                    """

                    // Clean up local tar file after deployment
                    sh "rm -f publish.tar"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful! Application is live on Azure App Service.'
        }
        failure {
            echo '❌ Deployment Failed! Check the logs for details.'
            sh "az webapp log tail --name $APP_SERVICE_NAME --resource-group $RESOURCE_GROUP"
        }
    }
}
