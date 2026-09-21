pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Arsalenn/Timesheet-DevOps.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
    }
}
