pipeline {

    agent none // Don't use any global agent

    stages {

        stage('Build') {

            agent {

                docker {

                    image 'python:3.9'

                }

            }

            steps {

                sh 'python -m py_compile sources/add2vals.py sources/calc.py'

                stash(name: 'compiled-results', includes: 'sources/*.py*')

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

        stage('Approval') {

            steps {

                input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk melanjutkan ke tahap Deploy)'

            }

        }

         stage('Deploy') {

            agent {

                docker {

                    image 'python:3.9'

                    args '-u root'

                }

            }

            steps {

                sh 'pip install pyinstaller'

                sh 'pyinstaller --onefile sources/add2vals.py'

                sleep time: 1, unit: 'MINUTES'

                echo 'Pipeline has finished successfully.'

            }

            post {

                success {

                    archiveArtifacts 'dist/add2vals'

                }

            }

        }

    }

}