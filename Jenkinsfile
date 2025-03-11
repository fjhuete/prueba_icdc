pipeline {
    environment {
        IMAGEN = "fjhuete/icdc"
        USUARIO = 'DockerHub'
    }
    agent none
    stages {
        stage("test the project") {
            agent {
                docker {
                image "python:3"
                args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'main',url:'https://github.com/fjhuete/prueba_icdc.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r app/requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'pytest test_app.py'
                    }
                }
            }
        }
        stage('Creación de la imagen') {
            agent any
            stages {
                stage('Build') {
                    steps {
                        script {
                            newApp = docker.build "$IMAGEN:$BUILD_NUMBER"
                        }
                    }
                }
                stage('Deploy') {
                    steps {
                        script {
                            docker.withRegistry( '', USUARIO ) {
                                newApp.push()
                                newApp.push("latest")
                            }
                        }
                    }
                }
                stage('Clean Up') {
                    steps {
                        sh "docker rmi $IMAGEN:$BUILD_NUMBER"
                        }
                }
            }
        }
        stage ('Despliegue') {
            agent any
            stages {
                stage ('Despliegue en el VPS'){
                    steps{
                        sshagent(credentials : ['Pignite']) {
                        sh 'ssh -o StrictHostKeyChecking=no debian@pignite.javihuete.site "cd prueba_icdc && git pull && docker compose down && docker pull fjhuete/icdc:latest && docker compose up -d && docker image prune -f"'
                        }
                    }
                }
            }
        }
    }    
    post {
        always {
        mail to: 'fjhuete.m@gmail.com',
        subject: "Despliegue de $IMAGEN: ${currentBuild.fullDisplayName}",
        body: "El despliegue ${env.BUILD_URL} de $IMAGEN ha tenido como resultado: ${currentBuild.result}"
        }
    }
}