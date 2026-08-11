pipeline {

    agent any

    tools {

        maven 'maven'

    }

    environment {
        TENANT_ID="be79d372-8667-48c7-9322-f8f44daef65e"
        SUBSCRIPTION_ID="7d4f1aed-616e-48dc-8ee1-73d74131da9b"
        IMAGE_NAME="springboot-app"
        IMAGE_TAG="latest"
        ACR_NAME="springbootcontainerreg"
        ACR_LOGIN_SERVER="springbootcontainerreg.azurecr.io"
        FULL_IMAGE_NAME="$ACR_LOGIN_SERVER/$IMAGE_NAME:$IMAGE_TAG"
        RG_NAME="jenkins-rg"
        AKS_CLUSTER_NAME="demo-aks"
        DEPLOYMENT_NAME="springboot-app"
        NAMESPACE="default"
        EMAIL_FROM="niharika.k1818@gmail.com"
        EMAIL_RECIPIENTS="niharikabandari1997@gmail.com"
    }

    stages {
        stage('Check Out from Git') 
        {
            steps {
                git branch: 'prod' , url: 'https://github.com/NihaDolly/azure-evening-springbootjavapp.git'
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
    //    stage ('Sonar Analysis') {
    //     environment {
    //         SCANNER_HOME = tool 'sonar-scanner'
    //     }
    //     steps {
    //         withSonarQubeEnv('sonar-server') {
    //             sh '''${SCANNER_HOME}/bin/sonar-scanner \
    //             -Dsonar.organization=nihadolly \
    //             -Dsonar.projectName=azure-evening-springbootjavapp \
    //             -Dsonar.projectKey=NihaDolly_azure-evening-springbootjavapp \
    //             -Dsonar.java.binaries=. \
    //             '''
    //         }
    //     }
    //    }
       stage('maven package') {
        steps {
            sh 'mvn package'
        }
      }

    //   stage('Sonar Quality Gate') {
    //     steps {
    //         timeout(time: 1, unit: 'MINUTES') {
    //             waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
            
    //         }
    //     }
    //   }
    
      stage ('Build Docker Image') {
        steps {
            echo "Building Docker Image"
            sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
        }
      }
      stage ('azure login and to acr') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', passwordVariable: 'AZURE_PASSWORD', usernameVariable: 'AZURE_USERNAME')]) {
                
                    script {
                        echo "Logging into Azure"
                        sh '''
                        az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                        az account set --subscription $SUBSCRIPTION_ID
                        az acr login --name $ACR_NAME
                        '''
                    }
                }
            } 
      }
        stage ('Push Docker Image to ACR') {
            steps {
                echo "Pushing Docker Image to ACR"
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $FULL_IMAGE_NAME'
                sh 'docker push $FULL_IMAGE_NAME'
            }
    }
     stage ('Azure login and deploy to AKS ') {
        steps {
            withCredentials([usernamePassword(credentialsId: 'azure-acr-spn', passwordVariable: 'AZURE_PASSWORD', usernameVariable: 'AZURE_USERNAME')]) {
                
                    script {
                        echo "Logging into Azure"
                        sh '''
                        az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID
                        az account set --subscription $SUBSCRIPTION_ID
                        az aks get-credentials --resource-group $RG_NAME --name $AKS_CLUSTER_NAME --overwrite-existing
                        echo "===== YAML IMAGE JENKINS IS USING ====="
                        grep -n "image:" k8s/sprinboot-deployment.yaml
                        echo "===== APPLYING YAML ====="
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
}
