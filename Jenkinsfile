pipeline {
    agent {
        docker {
            image 'ubuntu:22.04'
            args '-u root'
        }
    }
    
    environment {
        BUILD_DIR = 'build'
        ARTIFACT_NAME = 'printer-fw-sim-demo.tar.gz'
    }
    
    stages {
        stage('Checkout from SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Yuvaraj031125/printer-fw-sim-demo.git',
                        credentialsId: 'github-creds'
                    ]]
                ])
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y cmake g++ cppcheck libgtest-dev lcov
                    cd /usr/src/gtest
                    cmake CMakeLists.txt
                    make
                    cp lib/*.a /usr/lib
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    mkdir -p ${BUILD_DIR}
                    cd ${BUILD_DIR}
                    cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="--coverage" ..
                    make
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '''
                    cd ${BUILD_DIR}
                    ctest --output-on-failure
                '''
            }
        }
        
        stage('Package Artifact') {
            steps {
                sh '''
                    tar -czf ${ARTIFACT_NAME} build/printer-fw-sim
                '''
            }
        }
        
        stage('Collect Coverage') {
            steps {
                sh '''
                    cd ${BUILD_DIR}
                    lcov --capture --directory . --output-file coverage.info
                    lcov --remove coverage.info '/usr/*' --output-file coverage.info
                    genhtml coverage.info --output-directory coverage_report
                '''
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '${ARTIFACT_NAME}, ${BUILD_DIR}/coverage_report/**', fingerprint: true
            }
        }
    }
    
    post {
        always {
            publishTestResults testResultsPattern: '${BUILD_DIR}/test_results.xml'
            cleanWs()
            echo 'Pipeline finished'
        }
    }
}
