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
        stage('Create docker environments') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh"""
                pip install -r requirements.txt
                pip install -r requirements-test.txt
                """
            }
        }
        stage('Unit tests') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh 'python3 -m pytest --junitxml results.xml tests/'
            }
            post {
                always {
                archiveArtifacts artifacts: 'results.xml', fingerprint: true
                junit 'results.xml'
                }
            }
        }
        stage('Coverage Report') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh"""
                python3 -m coverage run --source=. --omit=tests/* -m pytest tests
                python3 -m coverage report -m
                python3 -m coverage html
                """
            }
            post {
                always {
                    publishHTML(target:[
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage report'
                    ])
                }
            }
        }
        stage('Deliver') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHub',
                usernameVariable: 'username',
                passwordVariable: 'password')]) {
                    sh"""
                    docker login -u ${username} -p ${password}
                    docker build -t ${username}/nif-validator .
                    docker push ${username}/nif-validator
                    """
                }
            }
        }
        stage('Deployment') {
            steps {
                sshagent(credentials:['aws-rhel-key-20241125']) {
                    sh"""
                    ssh -o StrictHostKeyChecking=no redhat@35.158.137.26 'docker rm -f nif-validator && run -d --name nif-validator -p 8080:9046 sbrm77/nif-validator'
                    """
                }
            }
        }    
    }
}
