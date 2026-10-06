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
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    cd ${BUILD_DIR}
                    ctest --output-on-failure --parallel $(nproc)
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
                    cd ${BUILD_DIR}
                    cpack -G TGZ
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
                            FILE=\$(ls \${BUILD_DIR}/*.tar.gz | head -1)
                            echo "Found: \$FILE"
                            curl -v -X POST \
                              "https://api.telegram.org/bot\${TELEGRAM_TOKEN}/sendDocument" \
                              -F "chat_id=\${TELEGRAM_CHAT_ID}" \
                              -F "document=@\${BUILD_DIR}/MyGame-1.0.0-Linux.tar.gz" \
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