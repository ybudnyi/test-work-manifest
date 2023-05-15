pipeline {
    agent any
    environment {
    VERSION = '1.0'
    BRANCH = 'main'
    PRODUCT_REPO = "https://github.com/ybudnyi/test-work-image.git"
    VCS_TOKEN = credentials('github')
  }
options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '3'))
}
parameters{
        string(description: 'version of build image', name: 'TAG')
          }
    stages {
        stage('PULL CHARTS') {
            steps {
                git (branch: "${BRANCH}", credentialsId: "github", url: "${PRODUCT_REPO}")
            }
        }
        stage('SUBSTITUTE IMAGE TAAG') {
            steps {
                sh 'envsubst < Chart > Chart.yaml'
                sh 'cat Chart.yaml'
            }
        }
        stage('Lint HELM chart') {
            steps {
                sh 'echo "Template STAGE env"'
                if (fileExists('stage-values.yaml')) {
                    sh "helm template -f int-values.yaml int-${JOB_BASE_NAME} ."
                }
                echo "Template PROD env"
                if (fileExists('prod-values.yaml')) {
                    sh "helm template -f prod-values.yaml prod-${JOB_BASE_NAME} ."
                }
                if (!fileExists('stage-values.yaml') && !fileExists('prod-values.yaml')) {
                    sh "helm template ${JOB_BASE_NAME} ."
                }

            }
        }
        stage('Template HELM chart for int and prod env') {
            steps {
                sh 'echo "Template STAGE env"'
                if (fileExists('stage-values.yaml')) {
                    sh "helm template -f int-values.yaml int-${JOB_BASE_NAME} ."
                }
                echo "Template PROD env"
                if (fileExists('prod-values.yaml')) {
                    sh "helm template -f prod-values.yaml prod-${JOB_BASE_NAME} ."
                }
                if (!fileExists('stage-values.yaml') && !fileExists('prod-values.yaml')) {
                    sh "helm template ${JOB_BASE_NAME} ."

            }
        }
        stage('Push HELM chart to Registry') {
            steps {
                sh "helm package ."
                env.VERSION_NUMBER = "${VERSION}.${TAG}"
                env.CHART = "app-${env.VERSION_NUMBER}.tgz"
                withCredentials([usernamePassword(credentialsId: 'helm', passwordVariable: 'PASSWD', usernameVariable: 'USERNAME')]) {
                    sh "helm registry login registry-1.docker.io -u ${USERNAME} -p ${PASSWD}"
                    sh "helm push ${env.CHART} oci://registry-1.docker.io/docker"
                    }
                }
            }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}