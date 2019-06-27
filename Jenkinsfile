#!/usr/bin/env groovy

pipeline{
    agent {
        label 'slave01'
    }

    parameters {
        string(name: 'BUILD_APPROVER_EMAIL', defaultValue: 'escuetamichael@gmail.com', description: 'Build Approver Email')
        string(name: 'BUILD_APPROVER_NAME', defaultValue: 'Michael Escueta', description: 'Build Approver Name')
    }
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
                    sh "curl -O \"https://jfrog-v2.ibm-kapamilya-devops.com/artifactory/group2/artifact.group2.${params.parent_build_number}.tar.bz2\""
                    sh "tar -xjf artifact.group2.${params.parent_build_number}.tar.bz2"
                }
            }
        }
        stage('Production Upload (Amazon S3)') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS_STAGING_CREDENTIALS']]) {
                    s3Upload bucket: 'group2.ibm-kapamilya-devops.com', file: "kapamilya-chat", workingDir: 'dist', acl: 'PublicRead'
                    sh '''
                        /usr/bin/aws cloudfront create-invalidation --distribution-id E1KAFIBVBSUAER --paths '/*'
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Always on run :D'
        }
        success {
            echo 'Deployment to Production Success!'
            buildStatusNotif("${currentBuild.currentResult}")
            sendEmailNotif("${currentBuild.currentResult}")
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
           echo 'Deployment to Production Failed!'
           buildStatusNotif("${currentBuild.currentResult}")
           sendEmailNotif("${currentBuild.currentResult}")
        }
        changed {
            echo 'Things were different before...'
        }
    }
}

def buildStatusNotif(String status) {
    def this_message = "Project Chat - BUILD " + status + ". More info at: '${env.BUILD_URL}'"
    def colorCode = '#00FF00'

    if(status == 'SUCCESS') {
        colorCode = '#00FF00'
    } else {
        colorCode  = '#FF0000'
    }
    slackSend (color: colorCode, message: this_message)
}

def sendEmailNotif(String status) {
    def this_body = "Hi ${BUILD_APPROVER_NAME}, <p>Job <strong>${env.JOB_NAME} | ${env.BUILD_NUMBER}</strong> was a "+status+".</p><p> More info at: ${env.BUILD_URL}</p>"
    def this_subject = "Project Chat [Production Deployment] - " + status
    emailext mimeType: 'text/html', body: this_body, subject: this_subject, to: '${BUILD_APPROVER_EMAIL}'
}