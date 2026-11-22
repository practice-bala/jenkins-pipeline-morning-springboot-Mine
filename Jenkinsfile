pipeline {
    agent any

    tools {
        maven 'maven'
    }

    

stages {

    stage ('Checkout From Git') {
        steps {
            git branch: 'prod', url: 'https://github.com/practice-bala/jenkins-pipeline-morning-springboot-Mine.git'
            }
        }
    stage ('Maven Validate') {
        steps {
            echo "This is Maven Validate"
            sh 'mvn validate'
            }
        }
    stage ('Maven Test') {
        steps {
            echo "This is Maven Test"
            sh 'mvn test'
            }
        }
    stage ('Maven package') {
        steps {
            echo "This is Maven package"
            sh 'mvn package'
            }
        }
    }
}

