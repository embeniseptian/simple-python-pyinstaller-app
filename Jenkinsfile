properties([
    pipelineTriggers([
        pollSCM('H/2 * * * *') // Poll SCM setiap 2 menit
    ])
])

node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml' // dipindahkan ke dalam blok 'inside' agar dijalankan di dalam kontainer
        }
    }
    stage('Deliver') {
        docker.image('cdrx/pyinstaller-linux:python2').inside {
            sh 'pyinstaller --onefile sources/add2vals.py'
            archiveArtifacts 'dist/add2vals' // dipindahkan ke dalam blok 'inside' dan dihapus kondisi 'success'
        }
    }
}