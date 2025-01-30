properties([
    pipelineTriggers([
        pollSCM('H/2 * * * *')  // Poll SCM setiap 2 menit
    ])
])

node {
    // Menggunakan Docker sebagai agent
    docker.image('node:16-buster-slim').inside('-p 3000:3000') {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
    }
}