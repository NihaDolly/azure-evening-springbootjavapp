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
    //     stage(' Trivy Scan')
    //     {
    //         steps {
    //             echo "Trivy Scan Started"
    //             sh 'trivy fs --format table --output trivy-report.txt --severity HIGH,CRITICAL .'
    //             echo "Trivy Scan Finished"
    //         }
    //     }
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
    //         timeout(time: 5, unit: 'MINUTES') {
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
                        kubectl apply -f k8s/springboot-deployment.yaml
                        '''
                    }
                }
            }
            
        }
  }
}