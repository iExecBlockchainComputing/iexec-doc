Be a worker pool
================

A worker pool (also called scheduler) is the essential actor of the infrastructure. It will be in charge of distributing the works submitted by the users to the different workers that are connected to this pool.

Start a scheduler
-----------------

A scheduler is deployed using docker. For this, you need to have docker already installed on the machine. You can follow the instructions on the `docker website <https://docs.docker.com/install/>`_ to install it. Since all the services used by iExec run in docker, we will use docker-compose to start the scheduler and its related service. You can follow the instructions  on the `docker compose website <https://docs.docker.com/compose/>` to install it.

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

The different variables you see here ()
