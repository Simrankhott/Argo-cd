@Library('jenkins-shared-library@main') _

pipeline {
    agent any
    
    tools { 
        maven 'maven3'
    }
    
    stages {
        stage('Git-Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Simrankhott/Argo-cd.git'
            }
        }
        
        stage('Build-Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'mvn clean package'
            }
        }
        
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://3.111.32.138:9000/"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
                }
            }
        }
        
        stage("DockerBuild and Push") {
            steps {
                dockerBuild("my-cicd", "simrankhot")
            }
        }
        
        stage("Docker-CleanUP") {
            steps {
                dockerCleanup("my-cicd", "simrankhot")
            }
        }
        
        stage("Ansible Setup") {
            steps {
                sh 'ansible-playbook ${WORKSPACE}/server_setup.yml'
            }
        }
        
        stage("Create deployment") {
            steps {
                sh "kubectl create -f ${WORKSPACE}/kubernetes-configmap.yml"
            }
        }
        
        stage("wait_for_pods") {
            steps {
                sh 'sleep 200'
            }
        }
        
        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "${GIT_USER_NAME}@simran.com"
                        git config user.name "${GIT_USER_NAME}"
                        git add .
                        git commit -m "Updated deployment file"
                        git push origin main
                    '''
                }
            }
        }
        
        stage("wait_for_pods2") {
            steps {
                sh 'sleep 200'
            }
        }
        
        stage("rollback deployment") {
            steps {
                sh """
                    kubectl delete deploy my-cicd
                    kubectl delete svc my-cicd
                """
            }
        }
    }
}
