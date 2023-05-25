@Library('jenkins-shared-library@main') _

pipeline {
    agent any
  
    parameters {
        choice(
            name: 'action',
            choices: 'create\rollback',
            description: 'Create/rollback of the deployment'
        )
        string(
            name: 'ImageName',
            description: "Name of the docker build",
            defaultValue: "my-cicd"
        )
        string(
            name: 'ImageTag',
            description: "Name of the docker build",
            defaultValue: "v1"
        )
        string(
            name: 'AppName',
            description: "Name of the Application",
            defaultValue: "my-cicd"
        )
        string(
            name: 'docker_repo',
            description: "Name of docker repository",
            defaultValue: "simrankhot"
        )
    }
      
    tools{ 
        maven 'maven3'
    }
    
    stages {
      
        stage('Git-Checkout') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Simrankhott/Argo-cd.git'
            }
        }
      
        stage('Build-Maven') {
            when {
                expression { params.action == 'create' }
            }
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
                SONAR_URL = "http://3.110.48.77:9000/"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
                }
            }
        }

        stage("DockerBuild and Push") {
            when {
                expression { params.action == 'create' }
            }
            steps {
                dockerBuild("${params.ImageName}", "${params.docker_repo}")
            }
        }
    
        stage("Docker-CleanUP") {
            when {
                expression { params.action == 'create' }
            }
            steps {
                dockerCleanup("${params.ImageName}", "${params.docker_repo}")
            }
        }
    
        stage("Ansible Setup") {
            when {
                expression { params.action == 'create' }
            }
            steps {
                sh 'ansible-playbook ${WORKSPACE}/server_setup.yml'
            }
        }
    
        stage("Create deployment") {
            when {
                expression { params.action == 'create' }
            }
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
            when {
                expression { params.action == 'create' }
            }
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
                    kubectl delete deploy ${params.AppName}
                    kubectl delete svc ${params.AppName}
                """	       
            }
        }
    }
}
