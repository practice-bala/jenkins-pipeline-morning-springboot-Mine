pipeline {
    agent any

    tools {
        maven 'maven'
    }

stages {

    stage ('Checkout From Git') {
        steps {
            git branch: 'prod', url: 'https://github.com/bkrrajmali/jenkins-pipeline-morning-springboot.git'
            }
        }
    stage ('Maven Validate') {
        steps {
            echo "This is Maven Validate"
            sh 'mvn validate'
            }
        }
    }
}

