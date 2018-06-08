Be a worker pool
================

A worker pool (also called scheduler) is the essential actor of the infrastructure. It will be in charge of distributing the works submitted by the users to the different workers that are connected to this pool.

Start a scheduler
-----------------

A scheduler is deployed using docker. For this, you need to have docker already installed on the machine. You can follow the instructions on the `the docker website <https://docs.docker.com/install/>`_ to install it. Since all the services used by iExec run in docker, we will use docker-compose to start the scheduler and its related service. You can follow the instructions  on the `the docker compose website <https://docs.docker.com/compose/>`_ to install it.

You can find the docker-compose file used for deployed on this repo: TODO
The docker compose file is working out-of-the-box, so nothing should be modified in that file. Only the .env along this docker compose file should be modified to use the correct parameters.

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
    
TODO
