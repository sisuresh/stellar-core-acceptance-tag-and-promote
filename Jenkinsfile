pipeline {
    agent any
    parameters {
        string(name: 'IMAGE', defaultValue: '', description: 'stelar-core image to tag and promote to testing')
    }
    environment {
        IMAGE = "${params.IMAGE}"
        
        DEB_PACKAGE_NAME=  "stellar-core_" + """${sh(
                returnStdout: true,
                script: 'docker run ${IMAGE} stellar-core version | cut -d " " -f 2'
            ).trim()}""" + "_amd64"
    }
    stages {
        stage('Run') {
            steps {
                sh '''#!/bin/bash

                    set -e

                    echo "IMAGE = ${IMAGE}"
                    echo "DEB_PACKAGE_NAME = ${DEB_PACKAGE_NAME}"
                    
                    STELLAR_CORE_VERSION=$(docker run ${IMAGE} stellar-core version)
                    echo "checking ${STELLAR_CORE_VERSION}"

                    #example version string - stellar-core 13.0.0~v2test (c097a3fe36f108bcbb52e4995b81bfe617f3109a)
                    gitv="$(echo "${STELLAR_CORE_VERSION}" | cut -d " " -f 3 | tr -d '()')"

                    echo "checking if ${STELLAR_CORE_VERSION} --> ${gitv} is a parent of master"

                    if [[ -z "$gitv" ]]; then
                    echo "Could not detect git version"
                    exit 1
                    fi

                    rm -rf stellar-core
                    git clone git@github.com:stellar/stellar-core
                    cd stellar-core

                    prev=$(git rev-parse --verify ${gitv}^ )
                    cur=$(git rev-parse --verify ${gitv} )

                    set +e
                    frommaster=$(git rev-list master "^${prev}" | grep "${cur}" )
                    set -e

                    if [[ -n "$frommaster" ]]; then
                    echo "Tagging acceptance-test-pass with ${gitv}"
                    git branch -f acceptance-test-pass-v2 ${gitv}
                    git push origin acceptance-test-pass-v2
                    else
                    echo "FAILURE"
                    echo "NOT updating acceptance-test-pass branch automatically:
                    ${STELLAR_CORE_VERSION} is not a parent of master
                    run:
                    git branch -f acceptance-test-pass-v2 ${gitv}
                    git push origin acceptance-test-pass-v2
                    "
                    exit 1
                    fi
                '''
            }
        }
    }
    post {
       success {
           script {
                build job: 'stellar-core-promote-to-testing', wait: false, parameters: [string(name: 'PACKAGE', value: "${DEB_PACKAGE_NAME}")]
            }
       }
    }
}