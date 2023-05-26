@Library('jenkins-shared-library@main') _

pipeline {
    agent any

    parameters {
        choice(
            name: 'action',
            choices: ['create', 'rollback'],
            description: 'Create/rollback of the deployment'
        )
        string(
            name: 'ImageName',
            description: 'Name of the docker build',
            defaultValue: 'my-cicd'
        )
        string(
            name: 'ImageTag',
            description: 'Name of the docker build',
            defaultValue: 'v1'
        )
        string(
            name: 'AppName',
            description: 'Name of the Application',
            defaultValue: 'my-cicd'
        )
        string(
            name: 'docker_repo',
            description: 'Name of docker repository',
            defaultValue: 'simrankhot'
        )
    }

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
                SONAR_URL = 'http://3.111.32.138:9000/'
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}"
                }
            }
        }


        stage("DockerBuild and Push") {
            steps {
                dockerBuild("${params.ImageName}", "${params.docker_repo}")
                dockerBuild("${params.ImageName}", "${params.docker_repo}")
            }
        }

        stage("Docker-CleanUP") {
            steps {
                dockerCleanup("${params.ImageName}", "${params.docker_repo}")
                dockerCleanup("${params.ImageName}", "${params.docker_repo}")
            }
        }

        stage("Ansible Setup") {
            steps {
                sh 'ansible-playbook ${WORKSPACE}/server_setup.yml'
            }
        }

        stage("Create deployment") {
            steps {
                sh "kubectl apply -f ${WORKSPACE}/kubernetes-configmap.yml"
            }
        }

        stage("wait_for_pods") {
            steps {
                sh 'sleep 200'
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = 'Argo-cd'
                GIT_USER_NAME = 'Simrankhott'
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        git config user.email 'simran@gmail.com'
                        git config user.name 'Simran khot'
                        BUILD_NUMBER=\${BUILD_NUMBER}
                        sed -i 's/replaceImageTag/\${BUILD_NUMBER}/g' kubernetes-configmap.yml
                        git add kubernetes-configmap.yml
                        git commit -m 'Update deployment image to version \${BUILD_NUMBER}'
                        git push https://\${GITHUB_TOKEN}@github.com/\${GIT_USER_NAME}/\${GIT_REPO_NAME} HEAD:main
                    """
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
