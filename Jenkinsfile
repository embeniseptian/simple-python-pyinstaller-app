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



        stage('Deploy to AWS EC2') {
    agent any
    steps {
        script {
            // Step 1: Transfer source code ke EC2
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: 'MyEC2',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'sources/*.py',
                                remoteDirectory: '/home/ec2-user/app',
                                execCommand: ''
                            )
                        ]
                    )
                ]
            )

            // Step 2: Jalankan perintah build dan deploy di EC2
            sshPublisher(
                publishers: [
                    sshPublisherDesc(
                        configName: 'MyEC2',
                        transfers: [
                            sshTransfer(
                                execCommand: '''
                                    cd /home/ec2-user/app && 
                                    docker run --rm -v $(pwd):/app -w /app python:3.9 sh -c "
                                        pip install pyinstaller && 
                                        pyinstaller --onefile sources/add2vals.py
                                    " && 
                                    echo 'Build artifacts created successfully.' && 
                                    ls -l dist/
                                '''
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
                        configName: 'MyEC2',
                        transfers: [
                            sshTransfer(
                                sourceFiles: '/home/ec2-user/app/dist/add2vals',
                                remoteDirectory: 'dist',
                                execCommand: ''
                            )
                        ]
                    )
                ]
            )
            archiveArtifacts 'dist/add2vals'
        }
        failure {
            echo 'Build failed. Check the logs for more details.'
        }
    }
}
    }
}