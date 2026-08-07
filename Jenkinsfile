pipeline {

    agent any

    tools {

        maven 'maven'

    }

    environment {
        TENANT_ID="be79d372-8667-48c7-9322-f8f44daef65e"
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
        stage(' Trivy Scan')
        {
            steps {
                echo "Trivy Scan Started"
                sh 'trivy fs --format table --output trivy-report.txt --severity HIGH,CRITICAL .'
                echo "Trivy Scan Finished"
            }
        }
       stage ('Sonar Analysis') {
        environment {
            SCANNER_HOME = tool 'sonar-scanner'
        }
        steps {
            withSonarQubeEnv('sonar-server') {
                sh '''${SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.organization=nihadolly \
                -Dsonar.projectName=azure-evening-springbootjavapp \
                -Dsonar.projectKey=NihaDolly_azure-evening-springbootjavapp \
                -Dsonar.java.binaries=. \
                '''
            }
        }
       }
       stage('maven package') {
        steps {
            sh 'mvn package'
        }
      }

      stage('Sonar Quality Gate') {
        steps {
            timeout(time: 1, unit: 'MINUTES') {
                waitForQuality abortPipeline: true credentialsId: 'sonar'
                echo "Sonar Quality Gate Passed"
            }
        }
      }
      
    }
}