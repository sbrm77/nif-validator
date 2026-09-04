#!groovy

/*
*   nif-validaotr project
*   20260904 Sérgio Monteiro
*/


pipeline {
    agent { label 'linux' }
    environment {
        HOME = "${env.WORKSPACE}"
    }

    stages {
        stage('Setup') {
            steps {
                sh 'printenv'
            }
        }
    }
}
