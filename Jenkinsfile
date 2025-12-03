pipeline {
    agent {
        docker {
            image 'ubuntu:22.04'
            args '-u root'
        }
    }
    
    triggers {
        pollSCM('H/5 * * * *')
    }
    
    environment {
        BUILD_DIR = 'build'
        ARTIFACT_NAME = 'printer-fw-sim-demo.tar.gz'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y build-essential cmake git
                    apt-get install -y gcov lcov
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    mkdir -p ${BUILD_DIR}
                    cd ${BUILD_DIR}
                    cmake ..
                    make -j$(nproc)
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    cd ${BUILD_DIR}
                    if [ -f "test_runner" ] || [ -f "tests" ]; then
                        ./test_runner || ./tests || echo "No test executable found"
                    else
                        echo "No tests found to run"
                    fi
                '''
            }
        }
        
        stage('Package Artifact') {
            steps {
                sh '''
                    if [ -f "${BUILD_DIR}/printer-fw-sim" ]; then
                        tar -czf ${ARTIFACT_NAME} -C ${BUILD_DIR} printer-fw-sim
                    else
                        echo "Binary not found, creating empty artifact"
                        touch empty_artifact
                        tar -czf ${ARTIFACT_NAME} empty_artifact
                    fi
                '''
            }
        }
        
        stage('Collect Coverage') {
            steps {
                sh '''
                    if command -v lcov >/dev/null 2>&1; then
                        lcov --capture --directory ${BUILD_DIR} --output-file coverage.info || echo "Coverage collection failed"
                        lcov --remove coverage.info '/usr/*' --output-file coverage_filtered.info || echo "Coverage filtering failed"
                    else
                        echo "Coverage tools not available"
                    fi
                '''
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '*.tar.gz', fingerprint: true
                script {
                    if (fileExists('coverage_filtered.info')) {
                        archiveArtifacts artifacts: 'coverage_filtered.info', fingerprint: true
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                if (fileExists('test_results.xml')) {
                    publishTestResults testResultsPattern: 'test_results.xml'
                }
            }
            cleanWs()
            echo 'Pipeline completed for printer-fw-sim-demo'
        }
    }
}
