# Deployment Automation

To clone this repo ```cd``` to selected folder use `git clone`:

```sh
cd <selected folder>
git clone https://github.com/KuzoOleh/Devops-app-api.git
```

## Local Development

### Running project

Docker is required in order to run this project locally.

Follow the below steps to run a local development environment:

1. Ensure [Docker Desktop](https://www.docker.com/products/docker-desktop/) is installed on your machine

2. After cloning the project and ```cd``` to it in Terminal run the following:

```sh
docker compose up
```

3. Browse the project at [http://127.0.0.1:8000/api/health-check/](http://127.0.0.1:8000/api/health-check/)

### Creating superuser

Since the deployed project is written in Python with Django you need to create superuser in order to access DJango admin.

Follow the below steps:

1. Run the below command and follow the in terminal instructions:
2. 
```sh
docker compose run --rm app sh -c "python manage.py createsuperuser"
```

2. Browse the Django admin at [http://127.0.0.1:8000/admin] and login.

### Cleaning Storage

To clear all storage (database included) and start clean:

```sh
docker compose down --volumes
docker compose up
```

### AWS CLI

#### AWS CLI authentication

This project was deployed using AWS services and uses [aws-vault](https://github.com/99designs/aws-vault) to autenticate with the AWS CLI in the terminal.

To autenticate: 

```
aws-vault exec PROFILE --duration=<1,2,4 or 8 hours>
```

>Note: replace `PROFILE` with the name of the profile

To list profiles, run:

```
aws-vault list
```

#### Task exec

[ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html) is used for manually running commands directly on the running containers.

To get shell access to the `ecs` task:

```
aws ecs execute-command --region REGION --cluster CLUSTER_NAME --task TASK_ID --container CONTAINER_NAME --interactive --command "/bin/sh"
```

Replace the following values in the above command:

- `REGION`: The AWS region where the ECS cluster is setup.
- `CLUSTER_NAME`: The name of the ECS cluster.
- `TASK_ID`: The ID of the running ECS task which you want to connect to.
- `CONTAINER_NAME`: The name of the container to run the command on.

### Terraform commands

>Note: All commands that required to work with terraform should be run from the `infra/` directory of the project, and after authenticating with `aws-vault`

To run any Terraform command through Docker, use the syntax below:

```
docker compose run --rm terraform -chdir=TF_DIR COMMAND
```

Where `TF_DIR` is the directory containing the Terraform (`setup` or `deploy`) and `COMMAND` is the Terraform command (e.g. `plan`).

### GitHub Actions Variables

This section lists the GitHub Actions variables which need to be configured on the GitHub project.

> Note: This is only applicable if using GitHub Actions, if you're using GitLab, see [GitLab CI/CD Variables](#gitlab-cicd-variables) below.

If using GitHub Actions, variables are set as either **Variables** (clear text and readable) or **Secrets** (values hidden in logs).

Variables:

- `AWS_ACCESS_KEY_ID`: Access key for the CD AWS IAM user that is created by Terraform and output as `cd_user_access_key_id`.
- `AWS_ACCOUNT_ID`: AWS Account ID taken from AWS directly.
- `DOCKERHUB_USER`: Username for [Docker Hub](https://hub.docker.com/) for avoiding Docker Pull rate limit issues.
- `ECR_REPO_APP`: URL for the Docker repo containing the app image output by Terraform as `ecr_repo_app`.
- `ECR_REPO_PROXY`: URL for the Docker repo containing the proxy image output by Terraform as `ecr_repo_proxy`.

Secrets:

- `AWS_SECRET_ACCESS_KEY`: Secret key for `AWS_ACCESS_KEY_ID` set in variables, output by Terraform as `cd_user_access_key_secret`.
- `DOCKERHUB_TOKEN`: Token created in `DOCKERHUB_USER` in [Docker Hub](https://hub.docker.com/).
- `TF_VAR_DB_PASSWORD`: Password for the RDS database (make something up).
- `TF_VAR_DJANGO_SECRET_KEY`: Secret key for the Django app (make something up).

