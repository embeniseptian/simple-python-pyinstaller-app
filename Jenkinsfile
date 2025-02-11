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

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    // Transfer file hasil build ke EC2 instance
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'MyEC2', // Nama konfigurasi SSH yang Anda buat di Jenkins
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/add2vals', // File yang akan di-deploy
                                        removePrefix: 'dist', // Hapus prefix 'dist' saat mengirim ke remote
                                        remoteDirectory: '/path/to/remote/directory', // Direktori remote di EC2 instance
                                        execCommand: 'chmod +x /path/to/remote/directory/add2vals && /path/to/remote/directory/add2vals' // Perintah yang akan dijalankan setelah file di-transfer
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}