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
            agent any // Atau gunakan `node` untuk menentukan executor
            steps {
                script {
                    // Transfer file hasil build ke EC2 instance
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'MyEC2', // Nama konfigurasi SSH yang Anda buat di Jenkins
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'sources/*.py', // File yang akan di-deploy
                                        removePrefix: 'sources', // Hapus prefix 'dist' saat mengirim ke remote
                                        remoteDirectory: '/home/ec2-user/app/sources', // Direktori remote di EC2 instance
                                        execCommand: '''
                                            cd /home/ec2-user/app && 
                                            docker run --rm -v $(pwd):/app -w /app python:3.9 sh -c "
                                                pip install pyinstaller && 
                                                pyinstaller --onefile sources/add2vals.py
                                            " && 
                                            echo 'Build artifacts created successfully.' && 
                                            ls -l dist/
                                        ''' // Perintah untuk build menggunakan Docker di EC2
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
            post {
                success {
                    // Step 3: Transfer artifacts hasil build kembali ke Jenkins (opsional)
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'MyEC2', // Nama konfigurasi SSH di Jenkins
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: '/home/ec2-user/app/dist/add2vals', // Transfer hasil build
                                        remoteDirectory: 'dist', // Direktori tujuan di Jenkins
                                        execCommand: '' // Kosongkan karena kita hanya ingin mentransfer file
                                    )
                                ]
                            )
                        ]
                    )
                    archiveArtifacts 'dist/add2vals' // Archive artifacts di Jenkins
                }
            }
        }
    }
}