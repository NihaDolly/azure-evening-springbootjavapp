pipeline {

    agent any

    tools {

        maven 'maven'

    }

    environment {
        TENANT_ID="ec78375d-0db0-42cf-82a6-2e6403e95936"
        IMAGE_NAME = "sprinbootapp"
        IMAGE_TAG = "latest"
        ACR_NAME= 'springbootdockerreg'
        ACR_LOGIN_SERVER ='springbootdockerreg.azurecr.io'
        FULL_IMAGE_NAME = "${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
        RG  = 'kubernetes-mgmt-srv'
        NAME = 'azure-aks-hpa'
        DEPLOYMENT_NAME ="springboot-app"
        NAMESPACE="default"
        EMAIL_RECIPIENTS = "erwwtwqetwq@gmail.com"
        EMAIL_FROM = 'tewewtewtq@gmail.com'
    }

    stages {
        stage('Check Out from Git') 
        {
            steps {
                git branch: 'prod' , url: 'https://github.com/bkrrajmali/azure-evening-springbootjavapp.git'
            }
        }

        // stage('Maven Validate') 
        // {
        //     steps {
        //         sh 'mvn validate'
        //     }
        // }

            // stage('Maven Compile') 
            // {
            //     steps {
            //         sh 'mvn compile'
            //     }
            // }
        // stage('Maven Test') 
        // {
        //     steps {
        //         sh 'mvn test'
        //     }
        // }
        // stage('Maven Install') 
        // {
        //     steps {
        //         sh 'mvn install'
        //     }
        // }
        // stage(' Trivy Scan')
        // {
        //     steps {
        //         echo "Trivy Scan Started"
        //         sh 'trivy fs --format table --output trivy-report.txt --severity HIGH,CRITICAL .'
        //         echo "Trivy Scan Finished"
        //     }
        // }

        // stage('Sonar Analysis')
        // {
        //     environment {
        //         SCANNER_HOME = tool 'Sonar-scanner'
        //     }
        //   steps {
        //       withSonarQubeEnv('sonarserver') {
        //         sh '''${SCANNER_HOME}/bin/sonar-scanner \
        //         -Dsonar.organization=bkrrajmali \
        //         -Dsonar.projectName=springbootapp \
        //         -Dsonar.projectKey=springbootapp \
        //         -Dsonar.java.binaries=.
        //         '''
        //       }
        //     }
        // }
        stage('Maven Package') 
        {
            steps {
                sh 'mvn package'
            }
        }
    //     stage('Sonar Quality Gate') 
    //     {
    //         steps {
    //             timeout(time: 1, unit: 'MINUTES') {
    //                 waitForQuality abortPipeline: true, credentialsId: 'sonar'
    //                 echo "Sonar Quality Gate Finished"
    //         }
    //     }
    //   }
      stage ('Docker Build')
      {
        steps {
            script {
            echo "Build Docker Image"
            docker.build  ("${IMAGE_NAME}:${IMAGE_TAG}")
      
        }
      }
   }
   stage('Azure Login and to ACR')
   {
    steps {
        withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD')]) 
        {
        script {
            echo "Azure Login"
            sh '''
            az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID 
            az acr login --name $ACR_NAME
            '''
        }
      }
    }
   }
   stage ('Docker Push')
   {
    steps 
    {
        script {
            echo"Docker Image Push"
            sh '''
            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}
            docker push ${FULL_IMAGE_NAME}
            '''
        }
    }
   }
   stage('Azure Login and AKS Deployment')
   {
    steps {
        withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD')]) 
        {
        script {
            echo "Azure Login"
            sh '''
            az account set --subscription "202d4be6-e0dd-4b9e-84b7-e235d53271a8"
            az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID 
            az aks get-credentials --resource-group $RG --name $NAME --overwrite-existing
            kubectl apply -f k8s/sprinboot-deployment.yaml
            '''
        }
      }
    }
   }
   stage('Verify Deployment Rollout')
   {
    steps {
        script {
            echo "Checking rollout status of deployment ${DEPLOYMENT_NAME} in namespace ${NAMESPACE}"
            // kubectl rollout status blocks until the rollout completes or the timeout is hit,
            // and exits non-zero on failure -- that non-zero exit is what fails the stage/pipeline.
            sh """
            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=60s
            """
        }
    }
   }
  }
 
  post {
    success {
        script {
            echo "Deployment verified successfully. Sending success email via Brevo API."
            withCredentials([string(credentialsId: 'brevo-api-key', variable: 'BREVO_API_KEY')]) {
                sh """
                curl --fail -s -X POST https://api.brevo.com/v3/smtp/email \\
                  -H "api-key: \$BREVO_API_KEY" \\
                  -H "Content-Type: application/json" \\
                  -d '{
                    "sender": {"email": "${EMAIL_FROM}"},
                    "to": [{"email": "${EMAIL_RECIPIENTS}"}],
                    "subject": "SUCCESS: Jenkins Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    "textContent": "Good news!\\n\\nThe pipeline ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully, and the deployment ${DEPLOYMENT_NAME} rolled out successfully to AKS.\\n\\nBuild URL: ${env.BUILD_URL}"
                  }'
                """
            }
        }
    }
    failure {
        script {
            echo "Pipeline or deployment verification failed. Sending failure email via Brevo API."
            withCredentials([string(credentialsId: 'brevo-api-key', variable: 'BREVO_API_KEY')]) {
                sh """
                curl --fail -s -X POST https://api.brevo.com/v3/smtp/email \\
                  -H "api-key: \$BREVO_API_KEY" \\
                  -H "Content-Type: application/json" \\
                  -d '{
                    "sender": {"email": "${EMAIL_FROM}"},
                    "to": [{"email": "${EMAIL_RECIPIENTS}"}],
                    "subject": "FAILED: Jenkins Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    "textContent": "The pipeline ${env.JOB_NAME} build #${env.BUILD_NUMBER} FAILED.\\n\\nThis could be due to a build/deploy step failing, or the deployment ${DEPLOYMENT_NAME} failing to roll out successfully in AKS (check the Verify Deployment Rollout stage logs).\\n\\nBuild URL: ${env.BUILD_URL}\\nConsole Log: ${env.BUILD_URL}console"
                  }'
                """
            }
        }
    }
    always {
        echo "Pipeline finished with status: ${currentBuild.currentResult}"
    }
  }
}
 
