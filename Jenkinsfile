//Library común
//Métodos que contiene: auditTools, sendMail
library identifier: 'jenkins-demo-library.git@main',
        retriever: modernSCM([$class: 'GitSCMSource', remote: 'https://github.com/jfrecio0157/jenkins-demo-library.git'])

pipeline {
    agent any

    environment {
        RELEASE = '20.04'
        MAIL = 'jfrecio@gmail.com'
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

        stage('Test'){
            when { expression {env.BRANCH_NAME == 'main' || env.BRANCH.NAME == 'brV01R00F00'}}
            steps{
                echo "Testing release ${RELEASE}"
                script{
                //Para brV01R00F00 el resultado de los test es aleatorio
                    if (env.BRANCH_NAME == 'brV01R00F00') {
                        script {
                          if (Math.random () > 0.5) {
                            throw new Exception() }
                        }
                    }

                    writeFile file: 'test-results.txt', text: 'passed'
                    //Guardar el archivo al final del paso Test
                    stash includes: 'test-results.txt', name: 'testResult'
                }
            }
        }

        stage('Audit Tools'){
            steps {
                echo "Audit Tools"
                auditTools()
            }
        }

        //Con when delimito cuando se va a ejecutar este stage
        stage('Run on Main'){
            when {branch 'main'}
                steps {
                    echo "Solo ejecuto en branch main"
                }
        }
    }

    post {
        success {
            //Recuperar el archivo
            unstash 'testResult'
            echo "Step post release ${RELEASE}"
            archiveArtifacts artifacts: 'test-results.txt'
            script{
                sendMail.success (mail: "${MAIL}")
                }
        }
        failure {
            script{
                sendMail.failure (mail: "${MAIL}")
                }
        }
    }
}
