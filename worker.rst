Be a worker
===========

A worker is an essential actor of the infrastructure. It will be in charge of performing the works submitted by the users. It is part of a worker pool. A worker will be rewarded some RLC for every work it performs.

Start a worker
--------------

Workers are deployed using docker. For this, you need to have docker already installed on the machine. You can follow the instructions on the `docker website <https://docs.docker.com/install/>`_ to install it.

**From a script provided by the scheduler**

The scheduler you want to connect your worker to may already provide a ready to launch script for you to use. In that case you can simply launch that script. It will start a worker with the correct parameters to connect to that scheduler. The only parameter that should change for you is the path of the workers wallet.

**Manual launch**

The following command can be called on the machine where the worker will run:

.. code:: bash

   docker run -d --restart always \
	        --env SCHEDULER_DOMAIN=pool.example.iex.ec \
	        --env SCHEDULER_IP=13.59.215.30 \
		--env LOGIN=test \
		--env PASSWORD=password123 \
		--env LOGGERLEVEL=DEBUG \
		--env SHAREDAPPS=docker \
		--env TMPDIR=/PATH/TO/TEMPDIR \
		--env WALLETPASSWORD=walletpassword \
		-v /PATH/TO/WORKER/WALLET/wallet.json:/iexec/wallet/wallet_worker.json \
		-v /var/run/docker.sock:/var/run/docker.sock \
		-v /PATH/TO/TEMPDIR:/PATH/TO/TEMPDIR \
		iexechub/worker:X.Y.Z

where X.Y.Z is the version of the worker that should be used. The list of available versions can be checked on the `iexec dockerhub page <https://hub.docker.com/r/iexechub/worker/tags/>`_. **It should match the version of the scheduler**.

The following parameters can be used in the command:

================  ==============================================  ==========  =============
Parameter         Meaning                                         Mandatory   Default value
================  ==============================================  ==========  =============
SCHEDULER_DOMAIN  URL of the scheduler                            YES
SCHEDULER_IP      IP adress of the scheduler                      NO
LOGIN             login of the worker to connect to scheduler     NO           vworker
PASSWORD          password of the worker to connect to scheduler  NO           vworkerp
SHAREDAPPS        apps already installed in the worker            NO           
LOGGERLEVEL       logger level used by the worker                 NO           INFO
TMPDIR            temporary folder used by the worker             NO
WALLETPASSWORD    password of the wallet used by the worker       NO
================  ==============================================  ==========  =============

Regarding the volumes mounted with the -v option in the docker run command, they are mandatory, **if not defined the worker may not behave as expected**:

1. The option *-v /PATH/TO/WORKER/WALLET/wallet.json:/iexec/wallet/wallet_worker.json* is used for the worker to know which wallet to use.
2. The option *-v /var/run/docker.sock:/var/run/docker.sock* is to allow the worker to start new docker containers when performing tasks. 
3. The option *-v /PATH/TO/TEMPDIR:/PATH/TO/TEMPDIR* will be used to store all the results from the worker.

Supported CPU
-------------
* **x86**
* **x86_64**

Supported OS
------------
* **Linux**
* **MacOS**
* **Windows**
