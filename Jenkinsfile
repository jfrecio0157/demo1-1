//Library común
//Métodos que contiene: auditTools, sendMail
@Library('jenkins-demo-library') _

pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Nombre de la branch a construir')
    }


    environment {
        RELEASE = '20.04'
        MAIL = 'jfrecio@gmail.com'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: 'https://github.com/tu-usuario/tu-repo.git'
            }
        }

       stage('Build') {
            environment {
                LOG_LEVEL = 'INFO'
            }
            steps {
                echo "Construyendo la Branch:  ${params.BRANCH}"
                echo "Building release ${RELEASE} with log level ${LOG_LEVEL} ..."
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                bat 'build.bat'
                }
            }
        }

        stage('Test'){
            when {
                anyOf {
                    expression { params.BRANCH == 'main' }
                    expression { params.BRANCH == 'brV01R00F00' }
                }
            }
            steps{
                echo "Testing release ${RELEASE} en branch ${params.BRANCH}"
                script{
                //Para brV01R00F00 el resultado de los test es aleatorio
                    if (params.BRANCH == 'brV01R00F00') {
                        script {
                          echo "Dentro del script de Test"
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
            when {expression { params.BRANCH == 'main' }}
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
