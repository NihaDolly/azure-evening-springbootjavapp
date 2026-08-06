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
            az account set --subscription "202d4be6-e0dd-4b9e-84b7-e235d53271a8"
            az login --service-principal -u $AZURE_USERNAME -p $AZURE_PASSWORD --tenant $TENANT_ID 
            az acr login --name $ACR_NAME
            '''
        }
      }
    }
   }
  }
}