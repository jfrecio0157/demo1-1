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
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                bat 'build.bat'
                }
            }
        }

        stage('Test') {
            steps {
                  echo "Testing release ${RELEASE}"
                  script {
                    if (Math.random () > 0.5) {
                        throw new Exception() }
                  }

                  writeFile file: 'test-results.txt', text: 'passed'
                  //Guardar el archivo al final del paso Test
                  stash includes: 'test-results.txt', name: 'testResult'
            }
        }

        stage('Audit Tools'){
            steps {
                echo "Audit Tools"
                auditTools()
            }
        }
    }

    post {
        success {
            //Recuperar el archivo
            unstash 'testResult'
            echo "Step post release ${RELEASE}"
            archiveArtifacts artifacts: 'test-results.txt'

            sendMailSuccess()
        }
        failure {
            sendMailFailure()
        }
    }
}

void auditTools () {
   echo "Verificando herramientas instaladas..."
   echo "Version java: "
   bat 'java -version'
   echo "Version git: "
   bat ' git --version'
}

void sendMailSuccess () {
   echo "Envio correo Success"
   mail to: 'jfrecio@gmail.com',
   subject: "Build Exitosa: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
   body: "La build ha sido exitosa. Ver detalles en ${env.BUILD_URL}"
}

void sendMailFailure() {
    echo "Envio correo Failure"
    mail to: 'jfrecio@gmail.com',
    subject: "Build Fallida: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
    body: "La build ha fallado. Revisa Jenkins en ${env.BUILD_URL}"
 }
