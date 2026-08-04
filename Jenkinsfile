pipeline {

    agent any

    tools {

        maven 'maven'

    }

    environment {
        TENANT_ID="ec78375d-0db0-42cf-82a6-2e6403e95936"
    }

    stages {
        stage('Check Out from Git') 
        {
            steps {
                git branch: 'prod' , url: 'https://github.com/bkrrajmali/azure-evening-springbootjavapp.git'
            }
        }

        stage('Maven Validate') 
        {
            steps {
                sh 'mvn validate'
            }
        }
    }
}