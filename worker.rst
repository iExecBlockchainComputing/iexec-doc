Worker
======================

A worker is an essential actor of the infrastructure. It will be in charge of performing the works submitted by the users. It is part of a worker pool. A worker will be rewarded some RLC for every work it performs.

.. include:: prerequisites.rst

Your wallet must be charged with 0.4 ETH.

| The worker registration to the workerpool does not require any RLC but to have a chance to be involved in a deal by processing tasks, the worker should stake an amount of RLC.
| This amount is not fixed and the workerpool will give you the minimum amount needed.


Install and start a worker
--------------------------

Contact the worker pool to get all the needed parameters to join its pool.

You should find a list of worker pool
https://pools.iex.ec/


Pool manager must provide connection location information, authentication login and password.

The following command can be called on the machine where the worker will run:

.. code:: bash
		
	docker run -d --name "my-iexec-worker" \
           --hostname "my-iexec-worker" \
           --env "IEXEC_WORKER_NAME=my-iexec-worker" \
           --env "IEXEC_CORE_HOST=main-pool.iex.ec" \
           --env "IEXEC_CORE_PORT=18090" \
           --env "IEXEC_WORKER_WALLET_PATH=/iexec-wallet/wallet.json" \
           --env "IEXEC_WORKER_WALLET_PASSWORD=mypassw00rd" \
           -v /home/ubuntu/wallet.json:/iexec-wallet/wallet.json \
           -v /tmp/iexec-worker:/tmp/iexec-worker\
           -v /var/run/docker.sock:/var/run/docker.sock \
           iexechub/iexec-worker:3.X.X


Please get the lastest version available (3.X.X) `here <https://hub.docker.com/r/iexechub/iexec-core/tags>`_. Note that it must match the version of the scheduler.

Please note that all the values shown here are just given as an example, it should be adapted to the worker pool you are trying to join and to the machine on which the worker will run.

Here is the details for the different parameters used in the command:

:IEXEC_WORKER_NAME: Name of your worker on the workerpool dashboard
:IEXEC_CORE_HOST: Domain of the scheduler
:IEXEC_CORE_PORT: Port of the scheduler
:IEXEC_WORKER_BASE_DIR: Should match the tmp folder your mounting (-v /tmp/iexec-worker). Results of tasks will be stored in /tmp/iexec-worker/my-iexec-worker)
:IEXEC_GAS_PRICE_MULTIPLIER: Increase it will speed up transactions (default: 1.3)
:IEXEC_WORKER_OVERRIDE_BLOCKCHAIN_NODE_ADDRESS: Use a custom ethereum node here, otherwise the one given by the core will be used

Regarding the volumes mounted with the -v option in the docker run command, they are mandatory, **if not defined the worker may not behave as expected**:

1. The option *-v /home/ubuntu/wallet.json:/iexec-wallet/wallet.json* is used for the worker to know which wallet to use.
2. The option *-v /tmp/iexec-worker:/tmp/iexec-worker* will be used to store all the results from the worker.
3. The option *-v /var/run/docker.sock:/var/run/docker.sock* is to allow the worker to start new docker containers when performing tasks. 

**From a script provided by the scheduler**

The scheduler you want to connect your worker to may already provide a ready-to-launch script for you to use out-of-the-box. In that case you can simply launch that script. It will start a worker with the correct parameters to connect to that scheduler. The only parameter that should change for you is the path of the worker's wallet.

Wallet restriction
------------------

An exclusive wallet must be associated to your worker.
You should create as many wallet as workers you get per worker pool.

Supported CPU
-------------

* **x86**
* **x86_64**

Supported OS
------------

Since a worker is running in a docker container, it can be started in any OS that can support docker, so the following OS are supported:

* **Linux**
* **MacOS**
* **Windows**

