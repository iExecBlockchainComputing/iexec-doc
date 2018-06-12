Be a worker pool
================

A worker pool (also called scheduler) is the essential actor of the infrastructure. It will be in charge of distributing the works submitted by the users to the different workers that are connected to this pool.

Docker compose file
-------------------

A scheduler is deployed using docker. For this, you need to have docker already installed on the machine. You can follow the instructions on `the docker website <https://docs.docker.com/install/>`_ to install it. Since all the services used by iExec run in docker, we will use docker-compose to start the scheduler and its related service. You can follow the instructions  on the `the docker compose website <https://docs.docker.com/compose/>`_ to install it.

You can find the docker-compose file used for deployed `here <https://github.com/iExecBlockchainComputing/iexec-deploy/blob/master/scheduler/docker-compose.yml>`_.

The docker compose file is working out-of-the-box, so nothing should be modified in that file. Only the .env along this docker compose file should be modified to use the correct parameters. You can find the .env file `here <https://github.com/iExecBlockchainComputing/iexec-deploy/blob/master/scheduler/.env>`_.

In the docker compose file, you will find the following components:

**Database service**

The first service, mandatory for the scheduler to work is the database. In the docker compose file, the db looks like this.

.. code:: bash 

  db:
    image: mysql:5.7
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ADMIN_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DB_NAME}
      MYSQL_USER: ${MYSQL_USER_LOGIN}
      MYSQL_PASSWORD: ${MYSQL_USER_PASSWORD}
    volumes:
      - ${MYSQLDATA_FOLDER}:/var/lib/mysql
      - ${DATABASE_FOLDER}:/docker-entrypoint-initdb.d
    networks:
      - iexec-net
    restart: unless-stopped

The different variables you see here (MYSQL_ADMIN_PASSWORD, MYSQL_DB_NAME, MYSQL_USER_LOGIN, MYSQL_USER_PASSWORD, MYSQLDATA_FOLDER and DATABASE_FOLDER) are defined in the .env file and **should be modified**.

Here is the list of variables used for the database and their meaning:

====================  ===============================================================================  ==========
Parameter             Meaning                                                                          Mandatory 
====================  ===============================================================================  ==========
MYSQL_ADMIN_PASSWORD  Admin password of the mysql database                                             YES
MYSQL_DB_NAME         Database name to use                                                             YES
MYSQL_USER_LOGIN      Login to use to connect to the database                                          YES
MYSQL_USER_PASSWORD   Password to use to connect to the database                                       YES
MYSQLDATA_FOLDER      Folder on the host machine where the db will be persisted                        YES
DATABASE_FOLDER       Folder containing the sql script that will be triggered at the start of the db   YES
====================  ===============================================================================  ==========

**Scheduler service**

The main component is the scheduler service. In the docker compose file, it is defined as follow:

.. code:: bash

  scheduler:
    image: iexechub/server:${SCHEDULER_DOCKER_IMAGE_VERSION}
    container_name: iexecscheduler
    hostname: iexecscheduler
    ports:
      - 4321:4321
      - 4322:4322
      - 4323:4323
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ${CERTIFICATE_AND_PRIVATE_KEYS_REPO}:/iexec/keystore
      - ${SCHEDULER_PERSISTING_FOLDER}:/var/xwhep
      - ${SCHEDULERWALLETPATH}:/iexec/wallet/wallet_scheduler.json
    environment:
      - DBHOST=${DBHOST}
      - DBNAME=${MYSQL_DB_NAME}
      - DBUSER=${MYSQL_USER_LOGIN}
      - DBPASS=${MYSQL_USER_PASSWORD}
      - ADMINLOGIN=${ADMINLOGIN}
      - ADMINPASSWORD=${ADMINPASSWORD}
      - ADMINUID=${ADMINUID}
      - WORKERLOGIN=${WORKERLOGIN}
      - WORKERPASSWORD=${WORKERPASSWORD}
      - WORKERUID=${WORKERUID}
      - LOGGERLEVEL=${LOGGERLEVEL}
      - JWTETHISSUER=${JWTETHISSUER}
      - JWTETHSECRET=${JWTETHSECRET}
      - DELEGATEDREGISTRATION=${DELEGATEDREGISTRATION}
      - MAXFILESIZE=${MAXFILESIZE}
      - BLOCKCHAINETHENABLED=${BLOCKCHAINETHENABLED}
      - ETHNODE=${ETHNODE}
      - RLCCONTRACT=${RLCCONTRACT}
      - IEXECHUBCONTRACT=${IEXECHUBCONTRACT}
      - WALLETPATH=${SCHEDULERWALLETPATH}
      - WALLETPASSWORD=${SCHEDULERWALLETPASSWORD}
      - WORKERPOOLADDRESS=${WORKERPOOLADDRESS}
    networks:
      - iexec-net
    restart: unless-stopped
    
Like the database, the different variables used here are defined in the file .env and **should be modified**.

=================================  ===============================================================================  ==========
Parameter                          Meaning                                                                          Mandatory 
=================================  ===============================================================================  ==========
SCHEDULER_DOCKER_IMAGE_VERSION     Version of the scheduler image to use
CERTIFICATE_AND_PRIVATE_KEYS_REPO  Path of the folder for the certificate that should be used by the scheduler
SCHEDULER_PERSISTING_FOLDER        Path of the folder that will persist the results
SCHEDULERWALLETPATH                Path of the scheduler's wallet
DBHOST                             Host of the db to which the scheduler will connect
MYSQL_DB_NAME                      Name of the db to use by the scheduler
MYSQL_USER_LOGIN                   Login of the user to use by the scheduler
MYSQL_USER_PASSWORD                Password of the user to use by the scheduler
ADMINLOGIN                         Admin login for the scheduler
ADMINPASSWORD                      Admin password for the scheduler
ADMINUID                           Admin UID of the scheduler
WORKERLOGIN                        Login of the worker that will connect to the pool
WORKERPASSWORD                     Password of the worker that will connect to the pool
WORKERUID                          Worker UID that will connect to the pool
LOGGERLEVEL                        Log level to use in the scheduler's log
JWTETHISSUER                       Issuer of the Json web token
JWTETHSECRET                       Password of the Json web token
DELEGATEDREGISTRATION
MAXFILESIZE
BLOCKCHAINETHENABLED               Boolean to say if the blockchain is used or not by the scheduler
ETHNODE                            Address of the ETH node that the scheduler will use
RLCCONTRACT                        Address of the RLC contract
IEXECHUBCONTRACT                   Address of the iExechub contract
SCHEDULERWALLETPATH                Path of the scheduler's wallet
SCHEDULERWALLETPASSWORD            Password of the scheduler's wallet
WORKERPOOLADDRESS                  
=================================  ===============================================================================  ==========

**Grafana service**

.. code:: bash

  grafana:
    image: iexechub/grafana:${GRAFANA_DOCKER_IMAGE_VERSION}
    container_name: iexecgrafana
    ports:
      - "3000:3000"
    environment:
      - DBHOST=db
      - MYSQL_DB_NAME=${MYSQL_DB_NAME}
      - MYSQL_USER=${GRAFANA_SQL_LOGIN}
      - MYSQL_PASSWORD=${GRAFANA_SQL_PASSWORD}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASSWORD}
      - GRAFANA_HOST=${GRAFANA_HOST}
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_NAME=ViewerOrg
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
      - GF_ALLOW_SIGN_UP=false
      - GRAFANA_HOME_NAME=${GRAFANA_HOME_NAME}
      - GRAFANA_HOME_LOGO_WIDTH=${GRAFANA_HOME_LOGO_WIDTH}
      - GRAFANA_HOME_LOGO_HEIGHT=${GRAFANA_HOME_LOGO_HEIGHT}
      - GRAFANA_HOME_LOGO_PATH=${GRAFANA_HOME_LOGO_PATH}
    volumes:
      - grafana-data:/var/lib/grafana
      - grafana-logs:/var/log/grafana
      - grafana-etc:/etc/grafana
    networks:
      - iexec-net
    restart: unless-stopped

Like the other services, the different variables used here are defined in the .env file and **should be modified**.

============================  ===============================================================================  ==========
Parameter                     Meaning                                                                          Mandatory 
============================  ===============================================================================  ==========
GRAFANA_DOCKER_IMAGE_VERSION  Image version of grafana
MYSQL_DB_NAME                 Name of the database where grafana will get all the statistics
GRAFANA_SQL_LOGIN             Login of the grafana user
GRAFANA_SQL_PASSWORD          Password of the grafana user
GRAFANA_ADMIN_PASSWORD        Admin password used in grafana
GRAFANA_HOST                  Address of grafana
GRAFANA_HOME_NAME             Name used in grafana's front end
GRAFANA_HOME_LOGO_PATH        Path of the logo used in grafana's front end
GRAFANA_HOME_LOGO_WIDTH       Width of the logo used in grafana's front end
GRAFANA_HOME_LOGO_HEIGHT      Height of the logo used in grafana's front end
============================  ===============================================================================  ==========

Start a scheduler
-----------------

To start a scheduler, it is pretty straightforward since the scheduler can be started like any docker compose service, so it can be started using:

.. code:: bash

  docker-compose up -d
  
Please note that you need to make sure the scheduler has finished its start before starting any worker that will connect to the pool.

Stop a scheduler
----------------

In a similar fashion, the scheduler can be stopped with the following command:

.. code:: bash

  docker-compose down
  
Please note that once the scheduler is turned off, the workers will not work anymore.  
