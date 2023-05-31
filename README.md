# Jenkins Pipeline 

This repository contains a Jenkins pipeline script written in declarartive format for automating the deployment process of an application using Jenkins and other related tools.

## Prerequisites

Before running this pipeline, make sure you have the following prerequisites:

- Jenkins server configured and running.

- Jenkins plugins installed: Git, Docker, Maven.

- Docker engine installed on the Jenkins server with all permissions necessary to avoid errors.

- Maven installed on the Jenkins server.

- Access to a SonarQube instance for static code analysis.

- kubeadm installation and also correct permissions set for kubeconfig file.

- Ansible installed and inventory configured.

- All secrets must be created in jenkins before triggering pipeline.

## Pipeline Overview

This Jenkins pipeline script automates the following stages of the deployment process:

1. **Git-Checkout**: Checkout the main branch of the Git repository containing the application source code.

2. **Build-Maven**: Build the application using Maven.

3. **Build and Test**: Perform additional build steps and run tests.

4. **Static Code Analysis**: Run static code analysis using SonarQube using sonar-token for authentication.

5. **DockerBuild and Push**: Build and push Docker image to dockerhub.

6. **Docker-CleanUP**: Clean up Docker images.

7. **Ansible Setup**: Set up the server using Ansible playbook.

8. **Create deployment**: Apply Kubernetes configuration for the deployment.

9. **wait_for_pods**: Pause the pipeline execution to allow time for Kubernetes pods to start.

10. **Update Deployment File**: Update the Kubernetes deployment file with the new image version.

11. **wait_for_pods2**: Pause the pipeline execution to allow time for Kubernetes pods to start with the updated deployment.

12. **rollback deployment**: Rollback the deployment by deleting the Kubernetes deployment.


## Usage

To use this pipeline script, follow these steps:

1. Set up a Jenkins job using this repository as the source.


2. Configure the job with the required parameters:

- `action`: Choose between 'create' or 'rollback' to indicate the desired deployment action.

- `ImageName`: Specify the name of the Docker build.

- `ImageTag`: Specify the version/tag of the Docker build.

- `AppName`: Specify the name of the application.

- `docker_repo`: Specify the name of the Docker repository.


3. Configure Jenkins credentials:

- `github`: Credentials for accessing the Git repository.

- `sonarqube`: Credentials for SonarQube authentication.

- `GITHUB_TOKEN`: GitHub personal access token for pushing changes to the Git repository.


4. Save the Jenkins job configuration and run the job.

Make sure you have the necessary access and permissions for the configured tools and resources.

## Contributing

If you find any issues or have suggestions for improvements, feel free to open an issue or submit a pull request to this repository.

## License

This project is licensed under the [MIT License](LICENSE).

Feel free to customize this README file based on your specific pipeline script and deployment process.

# Screenshots

<img width="960" alt="pro55" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/96fb6143-bf8e-46b1-8422-a0286b0addfe">
<br>
<img width="960" alt="pro56" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/a4c83905-49ef-40a8-aa32-87f75aa9d13e">
<br>
<img width="960" alt="pro57" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/2d74b37c-803c-4e91-8853-8ab154c0ae77">
<br>
<img width="739" alt="pro58" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/e54d3b2d-438f-435e-b763-420117844788">
<br>
<img width="960" alt="pro59" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/62fc636c-c697-4727-a150-c2107740465d">
<br>
<img width="960" alt="Screenshot 2023-05-27 161210" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/b27a9e5b-381a-4d7e-8ad9-374de87c44c4">
<br>
<img width="960" alt="Screenshot 2023-05-27 161332" src="https://github.com/Simrankhott/Argo-cd/assets/91006102/8d8a8055-8630-4aab-b23c-32c01aee7146">
<br>


