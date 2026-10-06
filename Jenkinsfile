pipeline {
    agent any

    environment {
        BUILD_DIR = 'build'
        TELEGRAM_TOKEN  = credentials('TELEGRAM_TOKON')
        TELEGRAM_CHAT_ID = credentials('TELEGRAM_CHAT_ID')
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
                }
            }
        }
    }

    post {
        failure {
            echo 'Build failed! Check the logs above.'
            node('')
                withCredentials([
                    string(credentialsId: 'telegram-bot-token', variable: 'TELEGRAM_TOKEN'),
                    string(credentialsId: 'telegram-chat-id',  variable: 'TELEGRAM_CHAT_ID')
                ]) {
                    sh """
                        curl -v -X POST \
                          "https://api.telegram.org/bot\${TELEGRAM_TOKEN}/sendMessage" \
                          -d "chat_id=\${TELEGRAM_CHAT_ID}" \
                          -d "text=Build FAILED: \${JOB_NAME} #\${BUILD_NUMBER}"
                    """
                }
        }
        success {
            echo 'Pipeline complete — package is ready.'
        }
    }
}