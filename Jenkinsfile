pipeline {
    agent any

    environment {
        RELEASE = '20.04'
    }
    stages {
       stage('Build') {
            environment {
                LOG_LEVEL = 'INFO'
            }
            steps {
                echo "Building release ${RELEASE} with log level ${LOG_LEVEL} ..."
                bat 'chmod +x m2/demo1-1/build.sh'
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER, passwordVariable: 'GIT_PASS')]) {
                  sh '''
                    .m2/demo1-1/build.sh
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                  echo "Testing release ${RELEASE}"
                  writeFile file: 'test-results.txt', text: 'passed'
                  //Guardar el archivo al final del paso Test
                  stash includes: 'test-results.txt', name: 'testResult'
            }
        }
    }

    post {
        success {
            //Recuperar el archivo
            unstash 'testResult'
            echo "Step post release ${RELEASE}"
            archiveArtifacts artifacts: 'test-results.txt'

        }
    }
}