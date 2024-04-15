# local-dbt-setup
The goal of this repo is to build a duplicable repository that houses the configuration and instructions for deploying DBT on a local containerised Postgres database for use with their free 

# Rough plan
- Use content on setting up a local containerised Docker container from this article: https://medium.com/@jewelski/quickly-set-up-a-local-postgres-database-using-docker-5098052a4726
- Wire it up with content from this article: https://medium.com/@jewelski/configure-my-dbt-core-side-project-using-my-local-postgres-database-f31c998ab6f3
- Iron out any configuration issues.

# Instructions for setup
## Spinning up docker
- Edit the [default.env_copy](default.env_copy) into a `default.env` file in the root directory with all 5 of the fields required. Set your own usernames and passwords as required, or use the defaults if you are only running locally and have no interaction with internet enabled connections:
  - POSTGRES_USER
  - POSTGRES_PASSWORD
  - POSTGRES_DB
  - PGADMIN_DEFAULT_EMAIL
  - PGADMIN_DEFAULT_PASSWORD
- You can then run docker compose with the environment file by doing: `docker-compose --env-file ./default.env up -d` which starts the containers in the background and not your current terminal

## Create a Server in pgAdmin
Now it’s time to create a server in pgAdmin. A server represents a connection to a PostgreSQL database server. You need to create a server in pgAdmin in order to connect to and manage a PostgreSQL database.
1. Open the pgAdmin web interface in your web browser. (http://localhost:5050/)
2. Log in using the email and password that you set in the PGADMIN_DEFAULT_EMAIL and PGADMIN_DEFAULT_PASSWORD environment variables in the docker compose file.
3. Right-click on the Servers node, and select Register -> Server… from the context menu.
4. In the Register — Server dialog that appears, enter a name for the server in the Name field.
5. In the Connection tab, enter the following information:
```
Host name/address: the hostname or IP address of the machine where the PostgreSQL database is running. If you are running the PostgreSQL container on the same machine as the pgAdmin container, you can use postgres as the hostname.
Port: the port number where the PostgreSQL database is listening for connections. In the Docker Compose file I provided earlier, the PostgreSQL container is exposing port 5432, so you can use 5432 as the port number.
Maintenance database: the name of the database that you want to use for maintenance tasks. You can use the postgres database for this purpose.
Username: the username that you want to use to connect to the database. You can use the POSTGRES_USER environment variable that you set in the Docker Compose file.
Password: the password for the user that you want to use to connect to the database. You can use the POSTGRES_PASSWORD environment variable that you set in the Docker Compose file.
```
6. Click the Save button to create the server.

TODO Figure out how to test this database with a postgresql query. Somehow need to connect to it locally and run something like:
```
CREATE TABLE test (id INT, name VARCHAR(255));
INSERT INTO test VALUES (1, 'postgres');
INSERT INTO test VALUES (2, 'pg-admin');
INSERT INTO test VALUES (3, 'docker');
SELECT * FROM test;
```

## Retrieving and running DBT for docker
Following the steps [here](https://docs.getdbt.com/docs/core/docker-install)
1. `docker pull ghcr.io/dbt-labs/dbt-core:latest`
2. `docker run --platform linux/arm64/v8 --name dbt-local -p 80:8080 -d ghcr.io/dbt-labs/dbt-core`
--COULDN'T GET PAST THIS STEP--
It appears there isn't support for containerised DBT on M1/M2 chips (i.e. only a linux/amd64 image exists)

Here are some other thoughts:

### Could try a Python install instead?
TODO - this is the consensus to try next

### Could try a direct binding after getting past previous step
There might be some guides out there on building in another system platform, maybe try: https://www.padok.fr/en/blog/docker-arm-architectures?
There might be some instructions in an earlier version of dbt-core such as in: https://github.com/dbt-labs/dbt-core/pkgs/container/dbt-core/197187118?tag=1.6.latest

The ENTRYPOINT for dbt Docker images is the command dbt. You can bind-mount your project to /usr/app and use dbt as normal:
```
docker run \
--network=host \
--mount type=bind,source=./,target=/usr/app \
--mount type=bind,source=./,target=/root/.dbt/ \
<dbt-image-name> \
ls
```

---
Resources:

https://docs.getdbt.com/docs/core/docker-install
https://www.stacksimplify.com/aws-eks/docker-basics/get-docker-image-from-docker-hub-and-run-/
