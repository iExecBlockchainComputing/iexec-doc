How to provide an application
=============================

| In this section we will show you how you can create a docker Dapp over iExec infrastructure.
| For developers, in the **task as a service** model, each time a task is launched through the iExec,
| they will be rewarded with the fee in your account.
| And you should then withdraw your funds at anytime to your own wallet.

Why using docker containers?
----------------------------

| A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.
| Docker Engine is the most widely used container engine. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.
| Docker is very convenient because it simplifies the deployment process, while ensuring consistency and repeatability in builds. Different people at different times will therefore build the same binary and obtain the same application behaviour.
| Another feature of Docker is the possibility of creating new layers that build on top of existing images. These existing images could be yours, or images proposed by the community.

https://docs.docker.com/storage/storagedriver/#images-and-layers

Build & test your docker image
------------------------------

Firstly you need to build a docker image that contains your application.

iExec supports linux-based Docker container.

Your image will be launched by iExec worker using following command:

.. code:: bash

   docker run -v ${IN_OUT}:/iexec ${DOCKERIMAGE} ${CMDLINE}



================  ==========================================================================================
Parameter         Meaning
================  ==========================================================================================
IN_OUT            directory used to exchange files with the container.
                  The files you put in `dirinuri` will be accessible in the container in the iexec folder.
                  The result of the application (as well as the consensus.iexec file)
                  should be in the iexec folder of the container. URL of the scheduler
DOCKERIMAGE       path to docker image to run
CMDLINE           command to execute for the application
================  ==========================================================================================

A list of applications with their docker images can be found at
https://github.com/iExecBlockchainComputing/iexec-apps


Deterministic result
--------------------

| iExec allows requester to ask for a result with a predefined level of trust.
| For the PoCo to run smoothly and verify that different workers return the same result, some determinism is needed at some point in the execution.
| Since it is not always easy (or even possible) to have exactly the same output of a job (for example, compute 3D rendering images on 2 different machines may produce 2 slightly different images).
| The PoCo will look for determinism of a file called **consensus.iexec**.
| This file **has to be created by the dapp and must be deterministic**.
| It can contain anything but the multiple runs of the job should produce exactly the same consensus.iexec file.
| If not, the PoCo will not find a consensus.

**Example:**

| Considering a application to blur faces on pictures,
| the content of the consensus.iexec file could simply be the coordinates of the faces in the pictures.
| The output of the execution (images with blur faces) may not be exactly the same, but the consensus.iexec file will be.

| `blur-face`: https://github.com/iExecBlockchainComputing/iexec-apps/tree/master/blur-face
| `find-face`: https://github.com/iExecBlockchainComputing/iexec-apps/tree/master/find-face


Put your image in Dockerhub
---------------------------

You must push your image to a public repository at DockerHub.
Before the executionof the task, iExec worker will pull the image from public repository.

Deploy your dapp
----------------

Once the application is available on docker, yous should register your application on the blockchain
and really create your decentralized and autonomous application, **a dapp**

.. code-block:: bash

    iexec app init

to set up the template in iexec.json

Then edit the iexec.json to describe your application: name, source, price,...

.. code:: javascript

  "app": {
    "name": "Ffmpeg",
    "price": 10,
    "params": {
      "type": "DOCKER",
      "envvars": "XWDOCKERIMAGE=jrottenberg/ffmpeg:scratch"
    }
  },

===================== =============================================
Parameter               Meaning
===================== =============================================
name                    dapp name
price                   price of your dapp in nRLC, i.e nanoRLC
app.params.type         type of dapp
app.params.envvars`     environment variables passed to your dapp
                        Do not remove "XWDOCKERIMAGE="
===================== =============================================

Then you deploy your dapp.

.. code-block:: bash

    iexec app deploy


Test your dapp
--------------

- Create a task template

.. code-block:: bash

    iexec order init
    ℹ using chain [kovan]
    ✔ Saved default order in "iexec.json", you can edit it:
    app:     0x0000000000000000000000000000000000000000
    dataset: 0x0000000000000000000000000000000000000000
    params:
      cmdline: --help

Edit the order part in iexec.json to describe your task

.. code:: javascript

  "order": {
    "buy": {
      "app": "0xXXXXXXXXXXXXXXXXXXX",
      "dataset": "0x0000000000000000000000000000000000000000",
      "params": {
        "cmdline": "-i /iexec/small.mp4 /iexec/small.avi",
        "dirinuri: "http://techslides.com/demos/sample-videos/small.mp4"
      }
    }


===================== ==========================================================
Parameter               Meaning
===================== ==========================================================
order.buy.app          Ethereum address where the application has been deployed
params.cmdline         command that will be executed in your container
params.dirinurifile    input downloaded to `/host` directory in docker container
                       , can be any type of file
                       , a zip archive will be decompressed automatically
===================== ==========================================================


Go to the `Getting started`_ section to learn how to test your dapp .

.. _Getting started: /gettingstarted.html
