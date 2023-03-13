## PostgreSQL for pybossa
The database setup is based on postgresql-pgadmin by [awesome-compose](https://github.com/docker/awesome-compose/tree/master/postgresql-pgadmin).  
It includes the following features:
- creation of permanent database with automatic initialization of pybossa user and database. Pybossa specific tables are **_not_** created.
- backup script for (daily) backups of pybossa database inside docker container, which can be added as cronjob
- pgAdmin web interface available at port 5050 (e.g. http://localhost:5050).

Project structure:
```
.
├── backup-db.sh
├── compose.yaml
├── config
│   ├── .env
│   ├── secrets
│   │   ├── pgadmin_pw.txt
│   │   ├── postgres_pw.txt
│   │   └── pybossa_pw.txt
│   └── servers.json
├── initdb
│   └── init-user-db.sh
├── [pgdata] ...
└── README.md
```

## Configuration

### .env
Before deploying this setup, you need to create and configure the following text files in the [_secrets_](config/secrets) folder.
- pgadmin_pw.txt: password for pgadmin user (username / email is defined in [_.env_](config/.env))
- postgres_pw.txt: password for default postgres user
- pybossa_pw.txt: relevant for initialization script – password for created user pybossa

This should be just the plain text password and look something like the following:
```
<password>
```
Please replace each placeholder <password> by a unique password.

### Bind Mounts
There are two bind mounts defined in [_compose.yaml_](compose.yaml) 
1. [initdb](initdb) hold the initialization scripts that are only executed on first container run
2. **pgdata** is the permanent store for the postgresql database on the host system and created on first container run. If the dir exists already and contains a functioning database then that database will be used and the initialization script will be skipped
 
## Deploy with docker compose
When deploying this setup, the pgAdmin web interface will be available at port 5050 (e.g. http://localhost:5050).  
On first deployment the [init-user-db.sh](initdb/init-user-db.sh) script will be executed and the **pgdata** dir is created as volume mount point for the postgresql database. 
This ensures that the contents of the database are not lost on shutting down the container.

``` shell
$ docker compose up -d
Starting postgres ... done
Starting pgadmin ... done
```
Please note: in case the database initialization process should be repeated the **pgdata** dir has to be deleted first. The [init-user-db.sh](initdb/init-user-db.sh) script is only executed if there exists no database already in pgdata! 

## Add postgres database to pgAdmin
After logging in with your credentials of the **pgadmin_pw.txt** file, you can open your database in pgAdmin, since it has been preconfigured by the [servers.json](config/servers.json). You just need to enter the password from **pybossa_pw.txt** when prompted
  
## Expected result

Check containers are running:
```
$ docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED             STATUS                 PORTS                                                                                  NAMES
849c5f48f784   postgres:latest                 "docker-entrypoint.s…"   9 minutes ago       Up 9 minutes           0.0.0.0:5432->5432/tcp, :::5432->5432/tcp                                              postgres
d3cde3b455ee   dpage/pgadmin4:latest           "/entrypoint.sh"         9 minutes ago       Up 9 minutes           443/tcp, 0.0.0.0:5050->80/tcp, :::5050->80/tcp                                         pgadmin
```

Stop the containers with
``` shell
$ docker compose down
# To delete all data run:
$ docker compose down -v
```
