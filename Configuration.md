

<div align="center">
   <h1>Wajir LIMS Deployment Guide</h1>
<img src="wajir_logo.png" alt="ProSync Logo" width="200"/>
<h3>Deployment Guide for Wajir LIMS</h3>
<p><em>Version 1.0 | May 2025</em></p>
</div>

## Table of Contents

1. [Introduction](#introduction)
2. [System Requirements](#system-requirements)
3. [Deployment Overview](#deployment-overview)
4. [Deployment Process](#deployment-process)
   - [Step 1: Docker Installation](#docker-installation)
   - [Step 2: Docker Compose Installation](#docker-compose-installation)
   - [Step 3: Application Deployment](#step-3-application-deployment)
   - [Step 4: Initial Data Setup](#step-4-initial-data-setup)
5. [System Backups and Maintenance](#system-backups-and-maintenance)
6. [System health Checks and Monitoring](#system-health-checks-and-monitoring)
7. [Troubleshooting](#troubleshooting)
8. [Appendix](#appendix)

## Introduction

This guide provides step-by-step instructions for deploying the Wajir LIMS application to a production environment using Vultr cloud servers. Wajir LIMS is a Land Information and Management System developed for Wajir County to 
for land administration, parcel management and revenue management from land related services.

The deployment process utilizes a mixture of docker commands and shell scripts to automate the setup of the server environment, installation of dependencies, application deployment, backup configuration, and monitoring setup.

## System Requirements

### Hardware Recommendations

- **CPU**: 4 vCPUs (minimum 2 vCPUs)
- **RAM**: 8GB (minimum 4GB)
- **Storage**: 160GB SSD (minimum 80GB)
- **Network**: Minimum 1Gbps network interface

### Software Requirements

- Ubuntu Server 24.04.2 LTS
- Docker
- Docker-Compose
- 

### Recommended Vultr Plan

- High Performance Cloud Compute
- 4 vCPU, 8GB RAM, 160GB SSD instance
- Ubuntu 24.04.2 x64

## Deployment Overview

The deployment process consists of the following major steps:

1. **Docker Installation**: Installation of Docker 
2. **Docker Compose Installation**: Installation of Docker Compose
3. **Application Deployment**: Deploy the LIMS application
4. **Initial Data Setup**: Populate the database with essential data
5. **Backup Configuration**: Set up automated backups
6. **Monitoring Setup**: Configure system monitoring


## Deployment Process

### Docker installation

Before commencing running or deploying the application, the server needs to be setup. Necessary plugins ought to be installed cognizant that this 
application will run in docker. Both Docker and Docker Compose ought to be installed in the system before proceeding.

### Install using the apt repository

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

#### Step 1: Set up Docker's apt repository.

```bash

#### Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```


```bash
#### Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
```

```bash
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt-get update
```
#### Step 2: Install the Docker packages.

Latest Specific version
To install the latest version, run:

```bash
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Step 3: Verify that the installation is successful by running the hello-world image:

```bash 
sudo docker run hello-world
```


### Docker Compose Installation

#### Installing Docker Compose
To make sure you obtain the most updated stable version of Docker Compose, you’ll download this software from its official Github repository.

First, confirm the latest version available in their releases page. At the time of this writing, the most current stable version is 1.29.2.

The following command will download the 1.29.2 release and save the executable file at /usr/local/bin/docker-compose, which will make this software globally accessible as docker-compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

Next, set the correct permissions so that the docker-compose command is executable:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

To verify that the installation was successful, you can run:

```bash
docker-compose --version
```

You’ll see output similar to this

```bash
  docker-compose version 1.29.2
```

```html
Output 
docker-compose version 1.29.2, build 5becea4c
```
### Step 3: Application Deployment

The deployment script handles the actual deployment of the LIMS application on the server.

```bash
docker-compose up --build -d
```

This docker command:
- Updates and upgrades system packages
- Installs essential utilities
- Downloads a postgis/postgis:latest image, which is a postgresql database with a postgis extension
- The Dockerfile ensures that the requirements and all dependency packages installed
- Installs pgbouncer for connection pooling
- Installs a database backup image that ensures data is backed based on the configurations passed
- Redis image installation and configuration
- Celery worker configuration
- tileserver installation and configuration
- Runs a script to prepopulate the database with subcounty and wards data
- Configures and sets up the Geoserver
- Sets timezone to Africa/Nairobi
- Creates a default admin user (email: admin@admin.admin , password:admin)
#### Environment Configuration

During deployment, an `.env` file will be created with default values. You should update this file with your specific configuration, especially API keys and credentials.

A template for this file is available at `/backend/env.template`.

#### NGINX Configuration

The deployment script sets up NGINX as a reverse proxy for the application. The NGINX configuration includes:

- HTTP to HTTPS redirection (if SSL is configured)
- Proxy settings for the backend,geoserver,tileserver
- Cache configuration for static assets
- Security headers
- Gzip compression

A template for this configuration is available at `/backend/Nginx/`.

### Step 4: Initial Data Setup

The data should be updated after the server is up and running since the default values auto-populates the database.

The following data should be updated

### IPN Settings

This setting is essential, it allows the payments to be done, without which the payment triggers fails.

##### Steps
- Login to the django_admin as a superuser/system administrator
- Search for the Payments and click on IPN Settings
- Add a default ipn setting to use
- The following fields will be required
  - Access Token Url
  - STK Push URL
  - Username
  - Password

**NB**: If you have more than one IPN setting, **ensure you check strictly one as the default IPN setting to avoid conflicts**.

### Notifications SetUp

The notification settings must be updated first for efficient processing and notifications within the system.

##### Steps
- Login to the django_admin as a superuser/system administrator
- Search for the Notification tab, click on email setup
- If more than two emails to be set, ensure one has been checked as the default to avoid conflict
  (recommended is one email to be setup)
- Pass the required fields
  - Sender email
  - password
  - Host
  - Port
  - Check if ```is_default``` email
  - Check ```use_tls``` to true

#### Finance Bill Fee

##### Steps
- Login to the django_admin as a superuser/system administrator
- Search for the Finance Bill Fee tab
- Select the FeeTypes ( At the point of documentation we have 20 fee types to update )
- Check and update the fees for each fee type based on the Finance Bill updates



## System Backups and Maintenance

The backup setup script configures automated daily backups for the LIMS application, including:

- Database backups
- Application code backups
- Media/File backups
- Optional remote backup to another server

To run the backup setup script:

### Run

To create a running container do:

```bash
docker run --name="backups" --hostname="pg-backups" --link db1:db -v backups:/backups -i -d kartoza/pg-backup:13.0
```

### Specifying environment variables
You can also use the following environment variables to pass a username and password etc for the database connection.

- POSTGRES_USER if not set, defaults to : docker
- POSTGRES_PASS if not set, defaults to : docker
- POSTGRES_PORT if not set, defaults to : 5432
- POSTGRES_HOST if not set, defaults to : db 
- POSTGRES_DBNAME if not set, defaults to : gis 
- ARCHIVE_FILENAME you can use your specified filename format here, default to empty, which means it will use default filename format. 
- DBLIST a space-separated list of databases to backup, e.g. gis postgres. Default is all databases. 
- REMOVE_BEFORE remove all old backups older than specified amount of days, e.g. 30 would only keep backup files younger than 30 days. Default: no files are ever removed.
- DUMP_ARGS='-Fc' The default dump argument to generate compressed database dumps. You can change this to generate other formats ie plain SQL dumps.
- RESTORE_ARGS='-j 4' The restore command to run four parallel jobs. You can specify other arguments based on official postgis_restore documentation.
- STORAGE_BACKEND='FILE' The default backend is to store the files on the host machine. Alternate backend is the s3 bucket (.ie minio or amazon bucket)
- DB_TABLES=yes Indicates if you need to dump all the tables in a DB into separate dumps. The default behaviour is not to show this so that the dumps are for the database.

#### Example usage:

```bash
 docker run -e POSTGRES_USER=bob -e POSTGRES_PASS=secret -link db -i -d kartoza/pg-backup
```

One other environment variable you may like to set is a prefix for the database dumps.

- DUMPPREFIX if not set, defaults to : PG

Here is a more typical example using [docker-compose](https://github.com/kartoza/docker-pg-backup/blob/master/docker-compose.yml)

### Filename format
The default backup archive generated will be stored in this directory (inside the container):

```
/backups/$(date +%Y)/$(date +%B)/${DUMPPREFIX}_${DB}.$(date +%d-%B-%Y).dmp
```

As a concrete example, with ```DUMPPREFIX=PG``` and if your postgis has DB name ```gis```. The backup archive would be something like:
```
/backups/2019/February/PG_gis.13-February-2019.dmp
```

If you specify ```ARCHIVE_FILENAME``` instead (default value is empty). The filename will be fixed according to this prefix. Let's assume ```ARCHIVE_FILENAME=/backups/latest``` The backup archive would be something like

```/backups/latest.gis.dmp```

### Backing up to S3 bucket
The script uses s3cmd⁠ to backup files to S3 bucket.

- ACCESS_KEY_ID= Access key for the bucket
- SECRET_ACCESS_KEY= Secret Access key for the bucket
- DEFAULT_REGION='us-west-2'
- HOST_BASE=
- HOST_BUCKET=
- SSL_SECURE='True' This determines if the S3 bucket is hosted with SSL site
- EXTRA_CONF= This is useful to add more configuration information to the s3cfg config file.
- BUCKET=backups Indicates the bucket name that will be created.


For a typical usage of this look at the docker-compose-s3.yml

### Restoring
A simple restore script is provided. You need to specify some environment variables first:

- TARGET_DB: the db name to restore
- WITH_POSTGIS: Kartoza specific, to generate POSTGIS extension along with the restore process
- TARGET_ARCHIVE: the full path of the archive to restore

**NB**: The restore script will try to delete the ```TARGET_DB``` if it matches an existing database, so make sure you know what you are doing. Then it will create a new one and restore the content from ```TARGET_ARCHIVE```

It is generally a good practice to restore into an empty new database and then manually drop and rename the databases. i.e if your original database was called ```gis``` you can restore into a new database called ```gis_restore```

If you specify these environment variables using docker-compose.yml file, then you can execute a restore process like this:

```bash
  docker-compose exec dbbackup /backup-scripts/restore.sh
```

## System health Checks and Monitoring

The system checks could occasionally be done to check on the health of different services running. We currently have two monitoring options.
1. Docker system health checks monitoring with portainer (simple, lightweight)
2. Advanced monitoring with Prometheus + Grafana (enterprise-grade)

To check on the status of different services in Wajir LIMS navigate to the following url.

Example

[Portainer Dashboard ](https://45.76.61.147:9443)

Ideal production access to the portainer dashboard as indicated below

[Portainer Dashboard Production]({{base_url}}/portainer/)

The ```base_url``` indicated above refers to the server 



The script will prompt you to choose your preferred monitoring solution and configure it accordingly.

Additionally, a health check script is created to monitor the status of critical services and alert system administrators if any issues are detected.


## Troubleshooting

### Common Issues

#### Application Not Starting

If there is a persistent issue with the application, down the application and restart it as follows

```bash
docker-compose down
```
Restart the application in detached mode

```bash
docker-compose up --build -d
```

If the issue persists, contact the LocateIT team for support.

Common causes include:
- Missing environment variables
- Database connection issues
- Port conflicts
- Celery worker down

#### Missing environment variables
If there are missing environment variables kindly, check on the ```env.production``` file and ensure the required variables have been passed.

For more information you could contact Locate or send and email to ```sirkipchumbamutai@gmail.com``` or call ```+254706019606```

#### Database Connection Issues

If the application cannot connect to the database, check:
- Database service status: `systemctl status postgresql`
- Database credentials in the `.env` file
- PostgreSQL configuration in `pg_hba.conf`

#### NGINX Configuration Issues

If NGINX is not properly proxying requests to the application, check:
- NGINX service status: `docker status nginx`
- NGINX configuration syntax: `docker nginx -t`
- NGINX error logs: `docker exec -it nginx && cat /var/log/nginx/error.log`

### Support

For additional support, contact:
- Locate IT Department: [evans.kipchumba@locateit.co.ke](mailto:evans.kipchumba@locateit.co.ke)
- Software Developer: [sirkipchumbamutai@gmail.com](mailto:sirkipchumbamutai@gmail.com)

## Appendix

### Script Reference

Incase the database is not prepopulated on starting the server.One could run still achieve by following the steps below.

Exec into the backend instance

```bash
docker exec -it backend bash
```

After running the above command, for one to run the python commands in shell they should run the below command

```bash
python3 manage.py shell
```

You could then go ahead with the running the scripts to prepopulate the database

| Script                                | Description                                        |
|---------------------------------------|----------------------------------------------------|
| `python3 manage.py makemigrations`    | checks if there is any changes made to the models  |
| `python3 manage.py migrate`           | Applies changes to the database                    |
| `python3 manage.py update_boundaries` | Populates the database with county & ward data     |


### File Locations

| File          | Description         |
|---------------|---------------------|
| `/backend/`   | Application code    |
| `/media/`     | Media files         |
| `/etc/nginx/` | NGINX configuration |
| `/backups/`   | Backup directory    |


### Environment Variables

| Variable                     | Description                                                              |
|-----------------------------|--------------------------------------------------------------------------|
| `BASE_URL`                  | Base URL for the application                                             |
| `DB`                        | Database service name                                                    |
| `CELERY_BROKER_URL`         | URL for the Celery broker (Redis)                                       |
| `CELERY_RESULT_BACKEND`     | Backend for storing Celery task results (Redis)                         |
| `DJANGO_SETTINGS_MODULE`    | Django settings module                                                   |
| `EMAIL_HOST_PASSWORD`       | Password for the email host                                              |
| `ENVIRONMENT`               | Environment type (e.g., PRODUCTION)                                      |
| `POSTGRES_DB`               | PostgreSQL database name                                                 |
| `POSTGRES_USER`             | PostgreSQL username                                                      |
| `POSTGRES_PASSWORD`         | PostgreSQL user password                                                 |
| `POSTGRES_HOST`             | PostgreSQL host                                                          |
| `POSTGRES_PORT`             | PostgreSQL port                                                          |
| `DOCKER_HOST`               | Docker socket path                                                       |
| `AUTH_TYPE`                 | Authentication type for PgBouncer (e.g., scram-sha-256)                 |
| `POOL_MODE`                 | Pool mode for PgBouncer (e.g., transaction)                              |
| `ADMIN_USERS`               | Admin users for PgBouncer                                                |
| `DB_NAME`                   | Alias for PostgreSQL database name                                       |
| `DB_USER`                   | Alias for PostgreSQL user                                                |
| `DB_PASSWORD`               | Alias for PostgreSQL user password                                       |
| `DB_HOST`                   | Alias for PostgreSQL host                                                |
| `DB_PORT`                   | Alias for PostgreSQL port                                                |
| `TZ`                        | Time zone configuration                                                  |
| `GDAL_LIBRARY_PATH`         | Path to the GDAL library                                                 |
| `GEOS_LIBRARY_PATH`         | Path to the GEOS library                                                 |
| `GEOSERVER_URL`             | URL to the GeoServer instance                                            |
| `GEOSERVER_DATA_DIR`        | Data directory for GeoServer                                             |
| `GEOSERVER_ADMIN_USER`      | Admin username for GeoServer                                             |
| `GEOSERVER_ADMIN_PASSWORD`  | Admin password for GeoServer                                             |
| `GEOSERVER_AUTH_ENABLED`    | Flag to enable GeoServer authentication                                  |
| `GEOSERVER_OPTS`            | GeoServer security configuration options                                 |
| `REST_SECURITY_ROLE`        | Role required to access GeoServer REST API                               |
| `CRON_SCHEDULE`             | Cron schedule for backups                                                |
| `REMOVE_BEFORE`             | Number of days to retain backups                                         |
| `GEOMETRY_SCHEMA`           | Schema name for storing geometry data                                   |
| `WORKSPACE_GEOSERVER`       | Workspace name for GeoServer                                             |
| `EMAIL_HOST_USER`           | Email address used as the sender                                         |
| `EMAIL_HOST_PASSWORD`       | Password for the email host (appears twice in config, choose the correct one) |
