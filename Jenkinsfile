node {
    def buildImage = 'python:2-alpine'
    def testImage = 'qnib/pytest'
    def deployImage = 'python:3.8-slim'

    stage('Checkout') {
        checkout scm
    }
    
    stage('Build') {
        docker.image(buildImage).inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage('Test') {
        docker.image(testImage).inside {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            } finally {
                junit 'test-reports/results.xml'
            }
        }
    }

    stage('Manual Approval') {
        try {
            timeout(time: 5, unit: 'MINUTES') {
                input message: 'Lanjutkan ke tahapan deploy?', ok: 'Proceed'
            }
        } catch (Exception e) {
            error 'Tidak dapat melanjutkan ke tahap deploy karena tidak ada persetujuan'
        }
    }

    stage('Deploy') {
        docker.image(deployImage).inside('--user root') {
            sh '''
                apt-get update && apt-get install -y binutils gcc python3-dev

                pip install pyinstaller

                pyinstaller --onefile sources/add2vals.py

                echo "Sleeping for 60 seconds..."
                sleep 60
            '''

            archiveArtifacts 'dist/add2vals'
        }
    }
}