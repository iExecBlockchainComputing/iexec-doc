Be a worker
===========

Start a worker in docker
------------------------

The easiest way to deploy a worker on a machine is to use docker. For this, you need to have docker already installed on the machine. You can follow the instructions on the `docker website <https://docs.docker.com/install/>`_ to install it.

**From a script provided by the scheduler**
The scheduler you want to connect your worker to may already provide a ready to launch script for you to use. In that case you can simply launch that script.

**Manual launch**
The following command can be called on the machine where the worker will run:

.. code:: bash

   docker run -d --restart always 
                --env SCHEDULER_DOMAIN="devxw.iex.ec" \
                -v /tmp:/tmp \
                -v /var/run/docker.sock:/var/run/docker.sock \
                iexechub/worker:X.Y.Z



Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.

Section 2
---------

Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.


Section 3
---------

Test test test
