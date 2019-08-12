# Secret

- Prerequisites

Enable docker swarm `docker swarm init`

### Securing Swarm With Secrets.

- We are going to create a `postgres` database and expose it using a database client `adminer`.The credentials will be provided as `secrets` instead of directly exposing it in docker-compose file.

- Create following secrets.

```command
echo "myuser" | docker secret create pg_user -
```
```command
echo "mysupersecretpassword" | docker secret create pg_password -
```
```command
echo "mydatabase" | docker secret create pg_database -
```
- List secret
```command
sudo dockerr secret ls
```
- To inspect a secret

```command
sudo docker inspect secret db_password
```
```output
[
    {
        "ID": "gr0e9l9kwe46vfhdaw7l0i04f",
        "Version": {
            "Index": 151
        },
        "CreatedAt": "2019-08-08T06:07:41.570783943Z",
        "UpdatedAt": "2019-08-08T06:07:41.570783943Z",
        "Spec": {
            "Name": "db_password",
            "Labels": {}
        }
    }
]
```


- Create a file `postgres-secret.yaml` as follows

```yaml
version: '3.1'
services:
    db:
        image: postgres
        restart: always
        environment:
            POSTGRES_USER_FILE: /run/secrets/pg_user
            POSTGRES_PASSWORD_FILE: /run/secrets/pg_password
            POSTGRES_DB_FILE: /run/secrets/pg_database
        secrets:
           - pg_password
           - pg_user
           - pg_database
    adminer: 
        image: adminer 
        ports: 
         - 8080:8080
secrets:
  pg_user:
    external: true
  pg_password:
    external: true
  pg_database:
    external: true
```
#### Details of the service file

- Docker secrets are stored as files in the path `/run/secrets`  of running containers and in above file we are pointing those files as environment variables.
- In the file we need to specify which are the secrets used for this service as follows.
- 
```yaml
secrets:
- pg_password
- pg_user
- pg_database
```
- Specify that the secrets are external

```yaml
secrets:
 pg_user:
   external: true
----
---
```
- Deploy the service using following command.

```command
docker stack deploy -c postgres-secrets.yml postgres

```
- Access the service using url `http://127.0.0.1:8080/`  and try to login using the credentials we provided as secret while creating the db.

- Details to login, select following details from dropdwon window.
  - system - postgreSQL
  - db - db
  
  
- Cleanup

```command
sudo docker stack down postgres
```

###  Creating a service with `secret` using `docker compose`

- Prerequesites

Install docker compose if not installed.

```command
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```command
sudo chmod +x /usr/local/bin/docker-compose
```
Verify the installation by running the following command 

```command
docker-compose --version
```
```output
docker-compose version 1.23.1, build b02f1306
```

#### Limitation of composing a service with secrets using `docker-compose`

- External secrets are not available to containers created by docker-compose. We need to create file with secret and specify it in docker-compose file
- Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.
- To deploy  application across the swarm, use `docker stack deploy`.(The method we used in previous example)

#### Deploying a wordpress application which utilizes `secret`

- Create  two secret file in project root directory as follows and add  credential in it.

```command
vim db_password.txt
```
```command
vim db_root_password.txt
```
The content of these two files will be utilized in docker compose file to create service.

- Create a `docker compose` file as follows.
```command
docker-compose.yaml
```

```yaml

version: "3.7"
services:
  db:
    image: mysql:latest
    command: "--default-authentication-plugin=mysql_native_password
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_root_password
      - db_password
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - published: 8089
        target: 80
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt
volumes:
  db_data:
```
*Here published port is used as `8089` to avoid conflict with an existing service, you can choose default port if needed*

- Specify that the secrets are accessed from the file which is in project root.

```yaml
secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt
```
- Deploy the service using `docker-compose` command.From the project root run the following.

```command
sudo docker-compose up -d
```
- Check if the containers are up and running.

```command
sudo docker ps
```
- Once the service is up , visit `http://127.0.0.1:8089`, you can see the wordpress site up with out asking for the password (we already passed thepassword as secret)

- To cleanup service.Following commands will stop the containers and will remove it
```command
docker-compose stop
```
```command
docker-compose rm
```
