# Conduit Server

This repository contains the configuration of the Conduit application with Docker. You can run it on your local machine or on your server. The Conduit application contains three single containers composed via Docker Compose: `conduit-frontend`, `conduit-backend` and `conduit-db`. The frontend is an Angular application, the backend is a Django application and the database is a Postgres database. This repository serves as an infrastructure repository which contains the submodules of the frontend and backend repositories.

The frontend and backend projects are forked from the following repositories:<br>
https://github.com/Developer-Akademie-GmbH/conduit-frontend<br>
https://github.com/Developer-Akademie-GmbH/conduit-backend<br>

<br>

## Table of contents

1. [Prerequisites](#prerequisites)
2. [Quickstart](#quickstart)
3. [Usage](#usage)
4. [Github Actions](#github-actions)
5. [Checklist](project_checklist_workflow.pdf)

<br>

## Prerequisites

-   Server with Docker and Docker Compose installed
-   Git installed to clone the repository

<br>

## Quickstart

This section describes how to run the Conduit server with Docker Compose on port `8282`. Only Docker Compose and Git are required to run the server.

1. Clone the repository with submodules:

```shell
git clone --recurse-submodules git@github.com:mikemeyer186/conduit.git
```

2. Change to directory `/conduit`:

```shell
cd conduit
```

> [!NOTE]
> If you already cloned the repository without submodules, you can initialize the submodules in the `/conduit` directory:

```shell
git submodule update --init --recursive
```

3. Create a `.env` file with the following content or copy the `example.env` file to `.env` and replace the unsafe values:

```shell
# Django
SECRET_KEY=<your_django_secret_key>
DJANGO_SUPERUSER_USERNAME=<your_admin_username>
DJANGO_SUPERUSER_EMAIL=<your_admin_email>
DJANGO_SUPERUSER_PASSWORD=<your_admin_password>

# Database
POSTGRES_USER=<your_db_user>
POSTGRES_PASSWORD=<your_db_password>
POSTGRES_DB=<your_db_name>
DB_NAME=<your_db_name>
DB_USER=<your_db_user>
DB_PASSWORD=<your_db_password>
DB_HOST=db     # This is the name of the service in the docker-compose.yaml file (do not change)
DB_PORT=5432   # This is the default port for Postgres (do not change)

# Deployment
VM_IP=<your_host_ip>
API_URL=<your_api_url>
```

4. Build and run the Docker Compose stack:

```shell
docker compose up -d
```

5. Stop the server:

```shell
# Stop the server, but keep the data saved
docker compose down

# or stop the server and remove all data
docker compose down -v
```

6. Open the Conduit application in the webbrowser:

```shell
# Angular Frontend
http://<your_host_ip>:8282

# Django Backend
http://<your_host_ip>:8020/admin
```

<br>

## Usage

The Conduit server is configured to run with some basic settings. You can configure these settings in the `docker-compose.yaml` file.

### Configuration

There are three services in the `docker-compose.yaml` file: `conduit-backend`, `conduit-frontend` and `conduit-db`. You can configure the following settings:

#### conduit-backend Service

-   `image`: The Dockerfile that is used for the service is stored in the `conduit-backend` submodule. The image which is used for the Django application is `Python 3.7.9-slim`.
-   `ports`: The ports that are exposed to the host. The default port is `8020`.
-   `networks`: The network that is used by both services containers. The default network is `conduit-network`.
-   `restart`: The restart policy for the container. The default policy is `unless-stopped`.

#### conduit-frontend Service

-   `image`: The Dockerfile that is used for the service is stored in the `conduit-frontend` submodule. The image which is used for the Angular application is `Node 20-slim` (Buildstage) and `Nginx 1.28-alpine-slim` (Productionstage).
-   `ports`: The ports that are exposed to the host. The default port is `8282`.
-   `networks`: The network that is used by both services containers. The default network is `conduit-network`.
-   `restart`: The restart policy for the container. The default policy is `unless-stopped`.
-   `build`: The build argument `API_URL` is used to set the API URL for the Angular application. This is required to connect the frontend with the backend. The default value is `http://<your_host_ip>:8020/api` which is saved in the `.env` file.

#### conduit-db Service

-   `image`: The image that is used for the service. The default image is `Postgres 13-alpine`.
-   `networks`: The network that is used by both services containers. The default network is `conduit-network`.
-   `restart`: The restart policy for the container. The default policy is `unless-stopped`.
-   `volumes`: The volume that is used to persist the data. The default volume is `db_data`.

### Logs

You can view the logs of the containers with the following command:

```shell
docker logs <container_name>

# or store it in a file
docker logs <container_name> > container-logs.txt

# or follow the logs in real time
docker compose logs -f <service_name>
```

> [!NOTE]
> I enabled error and access logs in the Nginx configuration of the frontend.

<br>

## Github Actions

This repository contains a Github Actions workflow that connects to your server, pulls the changes and starts the Docker containers. The workflow is triggered on every push to the specified branch in the workflow. The workflow is defined in the `.github/workflows/deploy.yml` file.

> [!NOTE]
> To use the Github Actions workflow, you need to set up the following secrets in your Github repository:

-   `SSH_PRIVATE_KEY`: The private SSH key to connect to your server.
-   `SSH_HOST`: The IP address of your server.
-   `SSH_PORT`: The port to connect to your server.
-   `SSH_USER`: The user to connect to your server.

### Generate SSH Key

1. Generate a new SSH key on your server with the following command. The public key will be saved in `~/.ssh/id_ed25519.pub` and the private key in `~/.ssh/id_ed25519`.

```shell
ssh-keygen -t ed25519 -a 200 -C "your_email@example.com"
```

2. Add the public key to your server's `~/.ssh/authorized_keys` file to allow access via SSH and add the private key to your Github repository secrets as `SSH_PRIVATE_KEY`.

### Configuration of Github Actions Workflow

You can configure the workflow in the `.github/workflows/deploy.yml` file. The following settings can be changed:

-   `branches`: The branch that is used to trigger the workflow. The default branch is `workflow`.
-   `DEPLOY_DIR`: The directory on your server where the repository is cloned. The default value is `$HOME/repos`.
