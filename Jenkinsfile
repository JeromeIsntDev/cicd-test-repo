/*
NOTES:
1. nproc => Command from CMake that is used to find all available processers
2. credentials => credentials that are from Jenkins (Found in 'Manage Jenkins -> Credentials')
*/

pipeline {
    agent any

    environment {
        BUILD_DIR = 'build'
        TELEGRAM_TOKEN  = credentials('tgram_token')
        TELEGRAM_CHAT_ID = credentials('tgram_chat_id')
    }

    stages {

        stage('Checkout') {
            steps {
                // Jenkins checks out the repo automatically for Pipeline jobs,
                // but this makes it explicit
                checkout scm
            }
        }

        stage('Configure (CMake)') {
            steps {
                sh '''
                    cmake -S . -B ${BUILD_DIR} \
                        -DCMAKE_BUILD_TYPE=Release \
                        -G "Unix Makefiles"
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    cmake --build ${BUILD_DIR} --parallel $(nproc)
                    stash includes: 'build/**', name: 'build-output'
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    cd ${BUILD_DIR}
                    ctest --output-on-failure --parallel $(nproc) || true
                '''
            }
            post {
                always {
                    // Archive CTest results if you generate XML (e.g. with --output-junit)
                    // junit '**/build/Testing/**/*.xml'
                    echo 'Tests complete.'
                }
            }
        }

        stage('Package') {
            steps {
                sh '''
                    unstash 'build-output'
                    sh 'cd ${BUILD_DIR} && cpack -G TGZ'
                '''
            }
            post {
                success {
                    // Archive the package so you can download it from Jenkins
                    archiveArtifacts artifacts: 'build/*.tar.gz', fingerprint: true
                    node('') {
                        sh """
                            echo "Looking for files in \${BUILD_DIR}:"
                            ls -la \${BUILD_DIR}/
                            FILE=\$(ls \${WORKSPACE}/\${BUILD_DIR}/*.tar.gz | head -1)
                            echo "Found: \$FILE"
                            curl -v -X POST \
                              "https://api.telegram.org/bot\${TELEGRAM_TOKEN}/sendDocument" \
                              -F "chat_id=\${TELEGRAM_CHAT_ID}" \
                              -F "document=@\${FILE}" \
                              -F "caption=✅ Build SUCCESS: ${JOB_NAME} #${BUILD_NUMBER}"
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Build failed! Check the logs above.'
            node(''){
                sh """
                    curl -v -X POST \
                      "https://api.telegram.org/bot\${TELEGRAM_TOKEN}/sendMessage" \
                      -d "chat_id=\${TELEGRAM_CHAT_ID}" \
                      -d "text=Build FAILED: ${JOB_NAME} #${BUILD_NUMBER}"
                """
            }
        }
        success {
            echo 'Pipeline complete — package is ready.'
        }
    }
}