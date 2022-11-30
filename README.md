
![Yamdb Workflow Status](https://github.com/t0pdog/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg?branch=master&event=push)

# CI and CD of the api_yamdb project. REST API Yamdb - database of user reviews about media works - books, films, music.

## Description
 
The YaMDb project collects user feedback on works. The works are divided into categories: "Books", "Films", "Music". The list of categories can be expanded (for example, you can add the category "Fine Arts" or "Jewellery"). The works themselves are not in YaMDb; you cannot watch a movie or listen to music here.
Configuration for Continuous Integration and Continuous Deployment application, implementation:
- automatic start of tests,
- updating images on Docker Hub,
- automatic deployment to the Yandex.Cloud combat server when pushing to the main main branch,
- sending a notification to the Telegram bot about the successful deployment of the project.

### User Roles
**Anonymous** - can view descriptions of works, read reviews and comments.

**Authenticated user (user)** - can read everything, like Anonymous, can additionally publish reviews and rate works (films / books / songs), can comment on other people's reviews and rate them; can edit and delete their reviews and comments.

**Moderator** - the same rights as an Authenticated User plus the right to delete and edit any reviews and comments.

**Administrator (admin)** - full rights to manage the project and all its contents. Can create and delete works, categories and genres. Can assign roles to users.

**Django Administrator** - Same rights as the Administrator role.

### Resources API YaMDb
**AUTH**: authentication.

**USERS**: users.

**TITLES**: Titles that are being reviewed (a specific movie, book, or song).

**CATEGORIES**: categories (types) of works ("Movies", "Books", "Music").

**GENRES**: genres of works. One work can be tied to several genres.

**REVIEWS**: reviews of works. The review is tied to a specific product.

**COMMENTS**: Comments on reviews. The comment is tied to a specific review.

## Technology stack:
[![Python](https://img.shields.io/badge/-Python-464646?style=flat&logo=Python&logoColor=56C0C0&color=008080)](https://www.python.org/)
[![Django](https://img.shields.io/badge/-Django-464646?style=flat&logo=Django&logoColor=56C0C0&color=008080)](https://www.djangoproject.com/)
[![Django REST Framework](https://img.shields.io/badge/-Django%20REST%20Framework-464646?style=flat&logo=Django%20REST%20Framework&logoColor=56C0C0&color=008080)](https://www.django-rest-framework.org/)
[![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-464646?style=flat&logo=PostgreSQL&logoColor=56C0C0&color=008080)](https://www.postgresql.org/)
[![JWT](https://img.shields.io/badge/-JWT-464646?style=flat&color=008080)](https://jwt.io/)
[![Nginx](https://img.shields.io/badge/-NGINX-464646?style=flat&logo=NGINX&logoColor=56C0C0&color=008080)](https://nginx.org/ru/)
[![gunicorn](https://img.shields.io/badge/-gunicorn-464646?style=flat&logo=gunicorn&logoColor=56C0C0&color=008080)](https://gunicorn.org/)
[![Docker](https://img.shields.io/badge/-Docker-464646?style=flat&logo=Docker&logoColor=56C0C0&color=008080)](https://www.docker.com/)
[![Docker-compose](https://img.shields.io/badge/-Docker%20compose-464646?style=flat&logo=Docker&logoColor=56C0C0&color=008080)](https://www.docker.com/)
[![Docker Hub](https://img.shields.io/badge/-Docker%20Hub-464646?style=flat&logo=Docker&logoColor=56C0C0&color=008080)](https://www.docker.com/products/docker-hub)
[![GitHub%20Actions](https://img.shields.io/badge/-GitHub%20Actions-464646?style=flat&logo=GitHub%20actions&logoColor=56C0C0&color=008080)](https://github.com/features/actions)
[![Yandex.Cloud](https://img.shields.io/badge/-Yandex.Cloud-464646?style=flat&logo=Yandex.Cloud&logoColor=56C0C0&color=008080)](https://cloud.yandex.ru/)

## Workflow
* tests - Check code in PEP8 standard (using flake8 package) and run pytest. Further steps will only be executed if the push was to the master or main branch.
* build_image_and_push_to_docker_hub - Build and push docker images to Docker Hub.
* deploy - Automatic deployment of the project to the server. Copying files from the repository to the server.
* send_message - Send notification to Telegram

### Preparing to run workflow
1. Create and activate virtual environment, update pip:
```
python3 -m venv venv
source venv/Scripts/activate
python3 -m pip install --upgrade pip
```
2. Run autotests:
```
pytest
```
3. Copy the prepared `docker-compose.yaml` file and the `nginx` folder with the configuration from your project to the server:
```
scp docker-compose.yaml <username>@<host>:/home/<username>/docker-compose.yaml
scp -r nginx <username>@<host>:/home/<username>/
```
4. In the GitHub repository, add data to `Settings - Secrets - Actions secrets`:
```
DOCKER_USERNAME - DockerHub username
DOCKER_PASSWORD - DockerHub user password
HOST - server ip address
USER - user on the server
SSH_KEY - private ssh key (must be public on the server)
PASSPHRASE - passphrase for ssh key (if created)
DB_ENGINE - django.db.backends.postgresql
DB_NAME - postgres (default)
POSTGRES_USER - postgres (default)
POSTGRES_PASSWORD - 12345 (default)
DB_HOST - db
DB_PORT - 5432
SECRET_KEY - the secret key of the django application (needs to be escaped or no brackets)
TELEGRAM_TO - id of your telegram account (can be obtained from @userinfobot, /start command)
TELEGRAM_TOKEN - bot token (you can get a token from @BotFather, /token, bot name)
```
### When making changes and executing commands:
```
git add .
git commit -m "..."
git push
```
The jobs command block is automatically executed (see Workflow)

## How to deploy the project locally:
All commands are executed in command line.

1. Clone the repository:
```
git clone https://github.com/t0pdog/yamdb_final.git
```
2. Change to the deployment folder:
```
cd yamdb_final/infra
```
3. Add a user to the docker group:
You can skip this step, in which case you must specify "sudo" at the beginning of each command
```
sudo usermod -aG docker username
```
4. Make sure the user is added to the group:
```
groups
```
5. Fill in the .envexample template:
It is best to use the built-in nano editor or any similar one.
```
nano .envexample
```
6. Fill in with secret data:
```
DB_ENGINE=django.db.backends.postgresql # database
DB_NAME=postgres # database name
POSTGRES_USER=postgres # login to connect to the database
POSTGRES_PASSWORD=12345 # password to connect to the database
DB_HOST=db # service (container) name
DB_PORT=5432 # port for connecting to the database
SECRET_KEY=12345 # secret key from the Django project
```
7. Rename env file:
```
cp .envexample .env
```
8. In the project folder, create an image:
Specify the username of your DockerHub account, image name and tag (optional).
```
docker build -t <username>/<imagename>:<tag>.
```
9. Build containers:
```
docker-compose -f infra/docker-compose.yaml up -d --build
```
or rebuild:
```
docker-compose up -d --build
```
10. Enter the container:
```
docker-compose exec web bash
```
10. Run the migrations:
```
python manage.py makemigrations
python manage.py migrate
```
11. Create a superuser:
```
python manage.py creates superuser
```
## How to deploy the project to the server:
1. Establish a connection to the server:
```
ssh username@server_address
```
2. Add a user to the docker group:
You can skip this step, in which case you must specify "sudo" at the beginning of each command
```
sudo usermod -aG docker username
```
3. Install Docker and Docker-compose:
```
apt install docker.io
apt-get update
apt-get install docker-compose-plugin
apt install docker-compose
docker-compose version
```
4. Create the `nginx` configuration on the local machine and transfer it to the server:
```
scp -r nginx username@ip_adress:/home/username/
```
5. Create a `docker-compose.yaml` file on the local machine and transfer it to the server:
```
scp docker-compose.yaml username@ip_adress:/home/username/
```
### After a successful deployment:
1. Display a list of running containers:
```
sudo docker container ls -a
```
2. In the list of containers, copy the CONTAINER ID of the username/api_yamdb:latest container (username is the username on DockerHub).
Log into the container:
```
sudo docker exec -it <CONTAINER_ID> bash
```
3. Inside the container, run the migrations:
```
python manage.py makemigrations
python manage.py migrate
```
4. Create a superuser:
```
python manage.py creates superuser
```
API documentation is available at http://ip_adress/redoc/
To access the documentation, you need to copy it to the statics folder:
```
cp redoc.yaml static/
```
The API is available at [http://ip_adress/api/v1/]
