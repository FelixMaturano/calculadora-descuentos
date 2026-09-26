// Practica 6 - Pipeline de calculadora-descuentos
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 15, unit: 'MINUTES')
    }

    triggers {
        //pollSCM('H/2 * * * *')
        githubPush()
    }

    stages {
        stage('Compilar') {
            steps {
                sh 'mvn -B clean compile'
            }
        }
        stage('Pruebas') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    // Publica el informe de JUnit aunque haya pruebas fallidas
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Cobertura') {
            steps {
                sh 'mvn -B verify'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/site/jacoco/**', allowEmptyArchive: true
                }
            }
        }
        stage('Análisis Estático') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn -B sonar:sonar -Dsonar.host.url=http://host.docker.internal:9000 -Dsonar.token=$SONAR_TOKEN'
                }
            }
        }
        stage('Empaquetar') {
            steps {
                sh 'mvn -B -DskipTests package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Build y pruebas exitosas'
        }
        failure {
            echo 'Fallo el proceso: revisa la etapa en rojo'
        }
    }
}
