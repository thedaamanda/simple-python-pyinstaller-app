node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
        }
    }
    stage('Deploy') {
        try {
            VOLUME = pwd() + "/${env.BUILD_ID}/sources:/src"
            IMAGE = 'cdrx/pyinstaller-linux:python2'

            dir(env.BUILD_ID) {
                unstash 'compiled-results'
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
            }

            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh 'sleep 60'
        } catch (e) {
            echo "Error: ${e}"
        } finally {
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }
    }
}
