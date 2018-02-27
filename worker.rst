Be a worker
===========

Supported CPU
-------------
* **x86**
* **x86_64**
* **ARM**

Supported OS
------------
* **Linux**
* **MacOS**
* **Windows**

Start a worker in docker
------------------------

The easiest way to deploy a worker on a machine is to use docker. For this, you need to have docker already installed on the machine. You can follow the instructions on the `docker website <https://docs.docker.com/install/>`_ to install it.

**From a script provided by the scheduler**

The scheduler you want to connect your worker to may already provide a ready to launch script for you to use. In that case you can simply launch that script. It will start a worker with the correct parameters to connect to that scheduler.

**Manual launch**

In case the scheduler does not provide such a script, the following command can be called on the machine where the worker will run:

.. code:: bash

   docker run -d --restart always \
                --env SCHEDULER_DOMAIN="devxw.iex.ec" \
                -v /tmp/iexec:/tmp/iexec \
                -v /var/run/docker.sock:/var/run/docker.sock \
                iexechub/worker:X.Y.Z

where X.Y.Z is the version of the worker that should be used. The list of available versions can be checked on the `iexec dockerhub page <https://hub.docker.com/r/iexechub/worker/tags/>`_. It should match the version of the scheduler.

The following parameters can be used in the command:

================  ==============================================  ==========  =============
Parameter         Meaning                                         Mandatory   Default value
================  ==============================================  ==========  =============
SCHEDULER_DOMAIN  URL of the scheduler                            YES
SCHEDULER_IP      IP adress of the scheduler                      NO
LOGIN             login of the worker to connect to scheduler     NO           vworker
PASSWORD          password of the worker to connect to scheduler  NO           vworkerp
SHAREDAPPS        apps already installed in the worker            NO           docker
SHAREDPACKAGES    packages already installed in the worker        NO
LOGGERLEVEL       logger level used by the worker                 NO           INFO
================  ==============================================  ==========  =============

Regarding the volumes mounted with the -v option in the docker run command, they are mandatory, if not defined the worker may not behave as expected:

1. The option *-v /tmp/iexec:/tmp* will be used to store all the results from the worker.
2. The option *-v /var/run/docker.sock:/var/run/docker.sock* is to allow the worker to start new docker containers. 

Install a worker from deb package 
---------------------------------

Coming soon.
