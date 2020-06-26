// Copyright (c) 2020, Oracle Corporation and/or its affiliates.
// Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl.

def HEAD_COMMIT

pipeline {
    options {
      skipDefaultCheckout true
      disableConcurrentBuilds()
    }

    agent {
        docker {
            image "${RUNNER_DOCKER_IMAGE}"
            args "${RUNNER_DOCKER_ARGS}"
            registryUrl "${RUNNER_DOCKER_REGISTRY_URL}"
            registryCredentialsId 'ocir-pull-and-push-account'
        }
    }

    environment {
        FLUENTD_MAJOR_VERSION = "v1.10"
        FLUENTD_MINOR_VERSION = "4"
        DOCKER_IMAGE_NAME = "fluentd:${FLUENTD_MAJOR_VERSION}.${FLUENTD_MINOR_VERSION}-oraclelinux-1.0-test"
        GOPATH = "$HOME/go"
        GO_REPO_PATH = "${GOPATH}/src/github.com/verrazzano"
        DOCKER_CREDS = credentials('ocir-pull-and-push-account')
    }

    stages {
        stage('Clean workspace and checkout') {
            steps {
                checkout scm
                sh """
                    echo "${DOCKER_CREDS_PSW}" | docker login ${env.DOCKER_REPO} -u ${DOCKER_CREDS_USR} --password-stdin
                    rm -rf ${GO_REPO_PATH}/fluentd-docker-image
                    mkdir -p ${GO_REPO_PATH}/fluentd-docker-image
                    tar cf - . | (cd ${GO_REPO_PATH}/fluentd-docker-image/ ; tar xf -)
                """
            }
        }

        stage('Build') {
	        when {
       		    not {
           	        branch 'master'
       	        }
   	        }
            steps {
                sh """
                    cd ${GO_REPO_PATH}/fluentd-docker-image/v1.10/oraclelinux
                    docker image build -t ${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${DOCKER_IMAGE_NAME} -f ./Dockerfile ../..
                    docker image push ${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${DOCKER_IMAGE_NAME}
                """
            }
        }

        stage('Scan Image') {
            when {
                not {
                   branch 'master'
                }
            }
            steps {
                script {
                    HEAD_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
                    clairScanTemp "${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${DOCKER_IMAGE_NAME}"
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/scanning-report.json', allowEmptyArchive: true
                }
            }
        }

    }

    post {
        failure {
            mail to: "${env.BUILD_NOTIFICATION_TO_EMAIL}", from: 'noreply@oracle.com',
            subject: "Verrazzano: ${env.JOB_NAME} - Failed",
            body: "Job Failed - \"${env.JOB_NAME}\" build: ${env.BUILD_NUMBER}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
        }
    }
}
