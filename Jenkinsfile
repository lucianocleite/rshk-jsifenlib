pipeline {
    agent any
    tools {
        jdk 'jdk8'
        maven "maven_3.6.3"
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    mvn clean install
                }
            }
        }
    }
}