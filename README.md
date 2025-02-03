# Deployment Automation

## Cloning the Repository

To clone this repository, navigate to your desired folder and run:

```sh
cd devops-app-api
git clone https://github.com/KuzoOleh/Devops-app-api.git
```

## Local Development

### Prerequisites

Ensure you have [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed on your machine before proceeding.

### Running the Project

Follow these steps to start the local development environment:

1. Clone the repository and navigate into the project directory:

    ```sh
    cd devops-app-api
    ```

2. Start the application using Docker Compose:

    ```sh
    docker compose up
    ```

3. Open your browser and navigate to:
   - [http://127.0.0.1:8000/api/health-check/](http://127.0.0.1:8000/api/health-check/)

### Creating a Superuser

Since the project uses Django, you need to create a superuser to access the Django admin panel:

1. Run the following command and follow the terminal instructions:

    ```sh
    docker compose run --rm app sh -c "python manage.py createsuperuser"
    ```

2. Access the Django admin panel at:
   - [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)

### Cleaning Storage

To remove all stored data (including the database) and start fresh:

```sh
docker compose down --volumes
docker compose up
```

## AWS CLI

### Authentication

This project is deployed using AWS services and uses [aws-vault](https://github.com/99designs/aws-vault) for authentication.

Authenticate with the AWS CLI by running:

```sh
aws-vault exec PROFILE --duration=<1,2,4 or 8 hours>
```

> **Note:** Replace `PROFILE` with your AWS profile name.

To list available profiles:

```sh
aws-vault list
```

### ECS Task Execution

[ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html) is used for running commands directly on containers.

To get shell access to an ECS task:

```sh
aws ecs execute-command \
  --region REGION \
  --cluster CLUSTER_NAME \
  --task TASK_ID \
  --container CONTAINER_NAME \
  --interactive \
  --command "/bin/sh"
```

Replace the placeholders:
- `REGION`: AWS region where the ECS cluster is located.
- `CLUSTER_NAME`: The ECS cluster name.
- `TASK_ID`: The running ECS task ID.
- `CONTAINER_NAME`: The container name.

## Terraform Commands

> **Note:** All Terraform-related commands must be run from the `infra/` directory **after authenticating with `aws-vault`**.

To run any Terraform command inside Docker:

```sh
docker compose run --rm terraform -chdir=TF_DIR COMMAND
```

Replace:
- `TF_DIR`: The Terraform directory (`setup` or `deploy`).
- `COMMAND`: The Terraform command (e.g., `plan`, `apply`).

## GitHub Actions Configuration

If using GitHub Actions, configure the following environment variables:

### Variables (Clear Text)
- `AWS_ACCESS_KEY_ID` – Access key for the AWS IAM user created by Terraform (`cd_user_access_key_id`).
- `AWS_ACCOUNT_ID` – Your AWS account ID.
- `DOCKERHUB_USER` – Docker Hub username to avoid pull rate limits.
- `ECR_REPO_APP` – URL for the app’s ECR Docker repository (`ecr_repo_app`).
- `ECR_REPO_PROXY` – URL for the proxy’s ECR Docker repository (`ecr_repo_proxy`).

### Secrets (Hidden in Logs)
- `AWS_SECRET_ACCESS_KEY` – Secret key corresponding to `AWS_ACCESS_KEY_ID`.
- `DOCKERHUB_TOKEN` – Personal access token for Docker Hub.
- `TF_VAR_DB_PASSWORD` – Password for the RDS database.
- `TF_VAR_DJANGO_SECRET_KEY` – Secret key for the Django application.
