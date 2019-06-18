#!/usr/bin/env groovy

pipeline{
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Deployment Started') {
            steps {
                echo "Deployment started..."
            }
        } 
        stage('Download Artifact (JFrog)') {
            steps {
                withCredentials([string(credentialsId: 'ARTIFACTORY_USERNAME_', variable: 'ARTIFACTORY_USERNAME_'), string(credentialsId: 'ARTIFACTORY_PASSWORD_', variable: 'ARTIFACTORY_PASSWORD_')]) {
                    sh '''
                        curl -O "https://jfrog-v2.ibm-kapamilya-devops.com/artifactory/group2/artifact.group2.$BUILD_NUMBER.tar.bz2"
                        tar -xjf artifact.group2.$BUILD_NUMBER.tar.bz2
                    '''
                }
            }
        }
        stage('Production Upload (Amazon S3)') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_STAGING_CREDENTIALS']]) {
                    s3Upload bucket: 'group2.ibm-kapamilya-devops.com', file: "kapamilya-chat", workingDir: 'dist', acl: 'PublicRead'
                }
            }
        }
    }
    post {
        always {
            echo 'Always on run :D'
        }
        success {
            echo 'Deploy Success!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
           echo 'Deploy Failed!'
        }
        changed {
            echo 'Things were different before...'
        }
    }
}
