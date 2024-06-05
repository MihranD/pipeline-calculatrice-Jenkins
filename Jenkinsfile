pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.8-alpine3.16'
                }
            }
            steps {
                sh 'python3.8 -m py_compile sources/prog.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'grihabor/pytest'
                }
            }
            steps {
                sh 'pytest -v --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit "test-reports/results.xml"
                }
            }
        }
        stage('Deliver') {
            agent any
            environment {
                VOLUME = "${WORKSPACE}/sources:/src"
                IMAGE = 'cdrx/pyinstaller-linux'
            }
            steps {
                dir("${WORKSPACE}/${env.BUILD_ID}") {
                    unstash 'compiled-results'
                    // Confirm files are in place
                    sh "ls /src"
                    // Run pyinstaller with correct paths
                    sh "docker run --platform linux/amd64 --rm -v ${VOLUME} ${IMAGE} pyinstaller -F /src/prog.py"
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: "${WORKSPACE}/${env.BUILD_ID}/sources/dist/prog", allowEmptyArchive: true
                    sh '''
                        chmod -R u+rwx ${WORKSPACE}/${env.BUILD_ID}/sources/build
                        chmod -R u+rwx ${WORKSPACE}/${env.BUILD_ID}/sources/dist
                        rm -rf ${WORKSPACE}/${env.BUILD_ID}/sources/build ${WORKSPACE}/${env.BUILD_ID}/sources/dist
                    '''
                }
            }
        }
    }
}
