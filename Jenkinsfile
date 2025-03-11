pipeline {
    environment {
        IMAGEN = "fjhuete/polls"
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
                        git branch:'master',url:'https://github.com/fjhuete/django_tutorial.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r requirements.txt'
                    }
                }
                stage('Test')
                {
                    steps {
                        sh 'python3 manage.py test --settings=django_tutorial.settings_desarrollo'
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
                        sh 'ssh -o StrictHostKeyChecking=no debian@pignite.javihuete.site "cd django_tutorial && git pull && docker compose down && docker pull fjhuete/polls:latest && docker compose up -d && docker image prune -f"'
                        }
                    }
                }
            }
        }
    }    
    post {
        always {
        mail to: 'fjhuete.m@gmail.com',
        subject: "Estado del pipeline: ${currentBuild.fullDisplayName}",
        body: "El despliegue ${env.BUILD_URL} ha tenido como resultado: ${currentBuild.result}"
        }
    }
}