pipeline {
    agent any

    options {
        skipDefaultCheckout()
        disableConcurrentBuilds()
    }

    tools {
        maven 'maven'
    }

    environment {
        GIT_URL = 'https://github.com/practice-bala/jenkins-pipeline-morning-springboot-Mine.git'
        GIT_BRANCH = 'prod'

        SONAR_SERVER = 'mine-sonarcloud'
        SONAR_SCANNER = 'mine-sonar-scanner'
        SONAR_ORGANIZATION = 'bkrrajmali'
        SONAR_PROJECT_KEY = 'bkrrajmali_mine-springbootapp'

        AWS_CREDENTIALS_ID = 'aws-jenkins'
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '175157388210.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'demoeks'
        IMAGE_URI = "${ECR_REGISTRY}/${ECR_REPOSITORY}:build-${env.BUILD_NUMBER}"

        EKS_CLUSTER = 'demoeks'
        K8S_NAMESPACE = 'default'
        K8S_MANIFEST = 'k8s/sprinboot-deployment.yaml'
        K8S_DEPLOYMENT = 'mine-springboot-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: env.GIT_BRANCH, url: env.GIT_URL
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn -B clean verify'
            }
        }

        stage('Sonar Analysis') {
            environment {
                SCANNER_HOME = tool(name: "${env.SONAR_SCANNER}")
            }
            steps {
                withSonarQubeEnv(env.SONAR_SERVER) {
                    sh '''
                        "$SCANNER_HOME/bin/sonar-scanner" \\
                            "-Dsonar.organization=$SONAR_ORGANIZATION" \\
                            "-Dsonar.projectKey=$SONAR_PROJECT_KEY" \\
                            "-Dsonar.java.binaries=target/classes"
                    '''
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                sh 'docker build -t "$IMAGE_URI" .'

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: env.AWS_CREDENTIALS_ID
                ]]) {
                    sh(script: '''
                        #!/usr/bin/env bash
                        set -euo pipefail

                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'rm -rf -- "$DOCKER_CONFIG"' EXIT

                        aws ecr get-login-password --region "$AWS_REGION" |
                            docker login --username AWS --password-stdin "$ECR_REGISTRY"

                        docker push "$IMAGE_URI"
                    '''.trim())
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: env.AWS_CREDENTIALS_ID
                ]]) {
                    sh(script: '''
                        #!/usr/bin/env bash
                        set -euo pipefail

                        kubeconfig_dir="$(mktemp -d)"
                        trap 'rm -rf -- "$kubeconfig_dir"' EXIT
                        export KUBECONFIG="$kubeconfig_dir/config"

                        aws eks update-kubeconfig --region "$AWS_REGION" --name "$EKS_CLUSTER"

                        sed "s|__IMAGE_URI__|$IMAGE_URI|g" "$K8S_MANIFEST" |
                            kubectl -n "$K8S_NAMESPACE" apply -f -

                        kubectl -n "$K8S_NAMESPACE" rollout status "deployment/$K8S_DEPLOYMENT" --timeout=5m
                    '''.trim())
                }
            }
        }
    }
}
