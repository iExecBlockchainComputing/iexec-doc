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

Deterministic result
--------------------
For the PoCo to run smoothly and verify that different workers return the same result, some determinism is needed at some point in the execution. Since it is not always easy (or even possible) to have exactly the same output of a job (for example, two images may be slightly different), the PoCo will look for determinism of a file called **consensus.iexec**. This file **has to be created by the dapp and must be deterministic**. It can contain anything but the multiple runs of the job should produce exactly the same consensus.iexec file. If not, the PoCo will not find a consensus. 

An example of such a file could be the following. Let's say a dapp is being develop to blur faces on pictures, the content of the consensus.iexec file could simply be the coordinates of the faces in the pictures given to the dapp. The output of the execution (images with blur faces) may not be exactly the same, but the consensus.iexec file will be.   

Some examples can be seen in the dapps:

* `blur-face`: https://github.com/iExecBlockchainComputing/iexec-apps/tree/master/blur-face
* `find-face`: https://github.com/iExecBlockchainComputing/iexec-apps/tree/master/find-face

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

