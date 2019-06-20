pipeline {
    agent none 
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
        stage('Lint') {
            agent {
                docker {
                    image 'eeacms/pylint'
                }
            }
            steps {
                sh 'pylint sources/test_calc.py --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" > pylint.log || exit 0'
                // || exit 0 is for ci we actually want --exit-zero but need newer version of pylint, make my own image?

            }
            post {
                always {
                    recordIssues enabledForFailure: true, tool: pyLint(pattern: 'pylint.log')
                }
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --cov sources --cov-report term-missing --cov-report xml --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                    cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'coverage.xml'
                }
            }
        }
        stage('Deliver') { 
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2' 
                }
            }
            steps {
                sh 'pyinstaller --onefile sources/add2vals.py' 
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals' 
                }
            }
        }
    }
}