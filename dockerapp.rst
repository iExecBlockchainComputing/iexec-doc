Docker Application
==================
In this section we will show you how you can create a docker Dapp over iExec infrastructure.

Build & test your image
-----------------------
Firstly you need to build a docker image that contains your application.

Your image will be launched by iExec worker using following command:

.. code:: bash

   docker run -v ${IN_OUT}:/iexec ${DOCKERIMAGE} ${CMDLINE}

* `IN_OUT` - directory used to exchange files with the container. The files you put in `dirinuri` will be accessible in the container in the iexec folder. The result of the application (as well as the consensus.iexec file) should be in the iexec folder of the container.
* `DOCKERIMAGE` - docker image to run
* `CMDLINE` - command to execute for the application

Push your image to Dockerhub
----------------------------
You must push your image to a public repository at DockerHub.
Before execution iExec worker will pull image from public repository.

Deploy your dapp
----------------
The easiest way to start building your dapp at iExec is to try our katacoda tutorial:
https://www.katacoda.com/sulliwane/scenarios/ffmpeg

In katacoda tutorial we are modifing some files in order to adapt Dapp Sample to your application.

Modifing ./iexec.js file
************************

.. code:: javascript

  "app": {
    "name": "Ffmpeg",
    "price": 10,
    "params": {
      "type": "DOCKER",
      "envvars": "XWDOCKERIMAGE=jrottenberg/ffmpeg:scratch"
    }
  },
  "order": {
    "buy": {
      "app": "0xXXXXXXXXXXXXXXXXXXX",
      "dataset": "0x0000000000000000000000000000000000000000",
      "params": {
        "cmdline": "-i /iexec/small.mp4 /iexec/small.avi",
        "dirinuri: "http://techslides.com/demos/sample-videos/small.mp4"
      }
    }
  }


* `name` - dapp name
* `app.params.type` - type of dapp
* `app.params.envvars` - environment variables passed to your dapp
  
.. warning:: It's very important to set XWDOCKERIMAGE variable. This variable sets the docker image of your dapp. 

* `order.buy.app` - Ethereum address where the application has been deployed
* `params.cmdline` - command that will be executed in your container
* `params.dirinuri` - file that will be downloaded to `/host` directory in docker container

  * Can be a single file
  * Can be `.zip` archive, which will be decompressed automatically

