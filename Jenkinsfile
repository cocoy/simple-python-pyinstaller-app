pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            environment { 
                VOLUME = '$(pwd)/sources:/src'
 		IMAGE = 'docker pull cdrx/pyinstaller-linux'
            }
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python'
                    args  '-v ${VOLUME}'
                }
            }
            steps {
                dir(path: env.BUILD_ID) { 
                    //unstash(name: 'compiled-results') 
	            sh 'pyinstaller -F add2vals.py'
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}
