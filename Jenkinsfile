pipeline {
    agent any
    environment {
        APP = 'app'
        VERSION = '1.0'
        BRANCH = 'main'
        PRODUCT_REPO = "https://github.com/ybudnyi/test-work-manifest.git"
        VCS_TOKEN = credentials('github')
        PATH="/usr/local/bin//build-dir:$PATH"
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
                sh '/usr/local/bin/envsubst < Chart > Chart.yaml'
                sh 'cat Chart.yaml'
            }
        }
        stage('Lint HELM chart') {
            steps {
                script {
                    sh 'echo "Template STAGE env"'
                    if (fileExists('stage-values.yaml')) {
                        sh "/usr/local/bin/helm template -f stage-values.yaml stage-${APP} ."
                    }
                    echo "Template PROD env"
                    if (fileExists('prod-values.yaml')) {
                        sh "/usr/local/bin/helm template -f prod-values.yaml prod-${APP} ."
                    }
                    if (!fileExists('stage-values.yaml') && !fileExists('prod-values.yaml')) {
                        sh "/usr/local/bin/helm template ${APP} ."
                    }
                }


            }
        }
        stage('Template HELM chart for int and prod env') {
            steps {
                script {
                    sh 'echo "Template STAGE env"'
                    if (fileExists('stage-values.yaml')) {
                        sh "/usr/local/bin/helm template -f stage-values.yaml stage-${APP} ."
                    }
                    echo "Template PROD env"
                    if (fileExists('prod-values.yaml')) {
                        sh "/usr/local/bin/helm template -f prod-values.yaml prod-${APP} ."
                    }
                    if (!fileExists('stage-values.yaml') && !fileExists('prod-values.yaml')) {
                        sh "/usr/local/bin/helm template ${APP} ."
                    }
                }

            }
        }
        stage('PUSH TO GIT') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: "ssh-git", keyFileVariable: 'key')]) {
                    sh 'git add .'
                    sh 'git commit -am "Updated version number"'
                    sh 'git push git@github.com:ybudnyi/test-work-manifest.git'
                    }

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