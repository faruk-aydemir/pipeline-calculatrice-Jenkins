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
                stash name: 'compiled-results', includes: 'sources/*.py*'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'grihabor/pytest'
                }
            }
            steps {
                sh 'pytest -v --junit-xml=test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Deliver') {
            agent any

            environment {
                IMAGE = 'cdrx/pyinstaller-linux'
            }

            steps {
                dir("${env.BUILD_ID}") {
                    unstash 'compiled-results'

                    sh '''
                    docker run --rm \
                      -v "$PWD/sources:/src" \
                      ${IMAGE} \
                      pyinstaller -F /src/prog.py
                    '''
                }
            }

            post {
                success {
                    archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/prog", fingerprint: true

                    sh """
                    rm -rf ${env.BUILD_ID}/sources/build
                    rm -rf ${env.BUILD_ID}/sources/dist
                    """
                }
            }
        }

    }
}