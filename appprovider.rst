Application provider
====================

| In this section we will show you how you can create a Docker dapp over the iExec infrastructure.
| In the **task as a service** model, each time a task is launched through the iExec network,
| Developers set the price of their app. Requesters pay on a pay-per-task basis.
| And you can then withdraw your funds at anytime to your own wallet.

Why using Docker containers?
----------------------------

| A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another.
| Docker Engine is the most widely used container engine. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.
| Docker is very convenient because it simplifies the deployment process, while ensuring consistency and repeatability in builds. Different people at different times will therefore build the same binary and obtain the same application behavior.
| Another feature of Docker is the possibility of creating new layers that build on top of existing images. These existing images could be yours, or images proposed by the community.

https://docs.docker.com/storage/storagedriver/#images-and-layers

Build & test your Docker image
------------------------------

We suppose your wallet is already created and charged with ETH to deploy your dapp and with RLC for testing.

Firstly you need to build a Docker image that contains your application.

iExec supports linux-based Docker container.

Your image will be launched by an iExec worker using following command.
If you application manages dataset, during the set up of your application, the dataset must be placed in the DIR_IN directory

.. code:: bash

   docker run -v ${DIR_IN}:/iexec_in -v ${DIR_OUT}:/iexec_out ${DOCKERIMAGE} ${CMDLINE}

.. WARNING::
    Use absolute path to define ${DIR_IN} and ${DIR_OUT} and not a relative path.

================  ==========================================================================================
Parameter         Meaning
================  ==========================================================================================
DIR_IN            directory where datasets is downloaded during the task initialization.
DIR_OUT           put all results files here. Full directory zipped in finalisation step.
                  The result of the application (as well as the determinism.iexec file)
                  should be in the iexec folder of the container. URL of the scheduler
DOCKERIMAGE       path to Docker image to run
ARGS              command to execute for the application
================  ==========================================================================================

A list of applications with their Docker images can be found at
https://github.com/iExecBlockchainComputing/iexec-apps


Deterministic result
--------------------

| iExec allows requesters to ask for a result with a predefined level of trust.
| For the PoCo to run smoothly and verify that different workers return the same result, some determinism is needed at some point in the execution.
| Since it is not always easy (or even possible) to have exactly the same output of a job (for example, compute 3D rendering images on 2 different machines may produce 2 slightly different images).
| The PoCo will look for determinism of a file called **determinism.iexec**.
| This file **has to be created by the dapp and must be deterministic**.
| It can contain anything but the multiple runs of the job should produce exactly the same determinism.iexec file.
| If not, the PoCo will not find a consensus.

**Example:**

| Considering a application to blur faces on pictures,
| the content of the determinism.iexec file could simply be the coordinates of the faces in the pictures.
| The output of the execution (images with blur faces) may not be exactly the same, but the determinism.iexec file will be.

| `blur-face`: https://github.com/iExecBlockchainComputing/iexec-apps/tree/master/blur-face
| `find-face`: https://github.com/iExecBlockchainComputing/iexec-apps/tree/master/find-face


How to manage datasets
----------------------

If the applications manages dataset,
the dataset is downloaded at the initialization of the task


You can test your application before the deploiement locally.

Create directories iexec_in iexec_out and put the dataset in iexec_in

.. code-block:: bash

      mkdir iexec_in iexec_out
      cp nsfw_model.zip iexec_in/.


Run and test locally your application with the following command

.. code-block:: bash

   docker run -e IEXEC_DATASET_FILENAME="nsfw_model.zip" -v `pwd`/iexec_in/:/iexec_in -v `pwd`/iexec_out:/iexec_out iexechub/nsfw_prediction:1.0 https://www.w3schools.com/w3css/img_lights.jpg



Put your image in Dockerhub
---------------------------

You must push your image to a public repository at DockerHub.
Before the execution of the task, iExec worker will pull the image from public repository.

.. note::
    Use docker tags mechanism to manage your application versioning.

    .. code-block:: bash

        docker tag iexechub/nilearn iexechub/nilearn:1.0
        docker push iexechub/nilearn:1.0


Deploy your dapp
----------------

Once the application is available on Docker, you have to register your application on the blockchain
and really create your decentralized and autonomous application, **a dapp**


Set up a configuration file.

.. code-block:: bash

    iexec init --skip-wallet
    iexec app init

to set up the template in iexec.json and fill information for registation: name, source, ...

===================== ==================================================
Parameter               Meaning
===================== ==================================================
owner                   the wallet address of the owner
name                    the dapp name
multiaddr               docker hub address of the application
checksum                "0x" + sha256 of the digest of the docker image
===================== ==================================================

Get the digest sha256:

.. code-block:: bash

    docker pull iexechub/nilearn:1.0
    1.0: Pulling from iexechub/nilearn
    Digest: sha256:f8a48dc5125fe762e3e35b9493291b8472a68782dd19d741a7e7aa062ef73dd6
    Status: Image is up to date for iexechub/nilearn:1.0

Don't forget the prefix "0x" for the checksum.

0xf8a48dc5125fe762e3e35b9493291b8472a68782dd19d741a7e7aa062ef73dd6


Edit iexec.json file,  set up the name, the docker address and the hash of the docker image
For a docker the checksum is obtained with a docker of the image

.. code:: javascript

  "app": {
    "owner": "0x47d0Ab8d36836F54FD9587e65125Bbab04958310",
    "name": "Nilearn",
    "type": "DOCKER",
    "multiaddr": "registry.hub.docker.com/iexechub/nilearn:1.0",
    "checksum": "0xf8a48dc5125fe762e3e35b9493291b8472a68782dd19d741a7e7aa062ef73dd6",
    "mrenclave": ""
  }


Then deploy the dapp.

.. code-block:: bash

    iexec app deploy --wallet-file developper_wallet
    ℹ using chain [kovan]
    ? Using wallet developper_wallet
    Please enter your password to unlock your wallet [hidden]
    ✔ Deployed new app at address 0xC97b068BffDf6Cf07C25d0Cfb01Bd079EebB134D


Publish app order
-----------------

Now the application registration is completed, let's publish an order to propose the application to the market

The application order will set the price, the volume and restriction.
Restriction are not mandatory.

- Create a order template

.. code-block:: bash

    iexec order init --app --wallet-file developper_wallet
    ℹ using chain [kovan]
    ✔ Saved default apporder in "iexec.json", you can edit it:
    app:                0xC97b068BffDf6Cf07C25d0Cfb01Bd079EebB134D
    appprice:           0
    volume:             1000000
    tag:                0x0000000000000000000000000000000000000000000000000000000000000000
    datasetrestrict:    0x0000000000000000000000000000000000000000
    workerpoolrestrict: 0x0000000000000000000000000000000000000000
    requesterrestrict:  0x0000000000000000000000000000000000000000


Sign the order

Edit the order part in iexec.json to describe your task,

===================== ==========================================================
Parameter               Meaning
===================== ==========================================================
 app                    app address
 appprice               app price
 volume                 number of order created, each usage decrease this number
 tag                    not use
 datasetrestrict:       restricted to a dataset  (1)
 workerpoolrestrict     restricted to a workerpool (1)
 requesterrestrict:     restricted to a requester (1)
===================== ==========================================================

(1) the restriction is disabled by default with 0x0000000000000000000000000000000000000000


.. code:: bash

    iexec order sign --app --wallet-file developper_wallet
    ℹ using chain [kovan]
    ? Using wallet developper_wallet
    Please enter your password to unlock your wallet [hidden]
    ✔ apporder signed and saved in orders.json, you can share it:
    app:                0xC97b068BffDf6Cf07C25d0Cfb01Bd079EebB134D
    appprice:           0
    volume:             1000000
    tag:                0x0000000000000000000000000000000000000000000000000000000000000000
    datasetrestrict:    0x0000000000000000000000000000000000000000
    workerpoolrestrict: 0x0000000000000000000000000000000000000000
    requesterrestrict:  0x0000000000000000000000000000000000000000
    salt:               0xda9180521bb3eb495e5fc9723d351199324b96481cdd85e9f7004477911045f0
    sign:               0xad835e8b86ccb9b44d3704fd64166da648927adf9dc88e96931de388033fb178192ee52a8c665fefe66b99296e299226d0f047aa8fb5bd87b7b165374154e3c51c

Publish the order

.. code:: bash

    iexec order publish --app --wallet-file developper_wallet
    ℹ using chain [kovan]
    ? Using wallet developper_wallet
    Please enter your password to unlock your wallet [hidden]
    ? Do you want to publish the following apporder?
    app:                0xC97b068BffDf6Cf07C25d0Cfb01Bd079EebB134D
    appprice:           0
    volume:             1000000
    tag:                0x0000000000000000000000000000000000000000000000000000000000000000
    datasetrestrict:    0x0000000000000000000000000000000000000000
    workerpoolrestrict: 0x0000000000000000000000000000000000000000
    requesterrestrict:  0x0000000000000000000000000000000000000000
    salt:               0xda9180521bb3eb495e5fc9723d351199324b96481cdd85e9f7004477911045f0
    sign:               0xad835e8b86ccb9b44d3704fd64166da648927adf9dc88e96931de388033fb178192ee52a8c665fefe6
    6b99296e299226d0f047aa8fb5bd87b7b165374154e3c51c
     Yes
    ✔ apporder successfully published with orderHash 0x2d09cc3e08e675fc290b683aa376b7038d1762f31674e97baaaa723a0e879fdc


Now the application is available.

Check out http://explorer.iex.ec


Go to the `Quick start`_ section to learn how to test your dapp .

.. _Quick start : /quickstart.html

Variables available in the container
------------------------------------

When a worker triggers the computation of a task, a few variables are available to the application that is running. They can be used by the application.

General variables
-----------------

Those variables are available in the container performing the computation of a task:

========================== ======================================================================================================
Variables                    Meaning
========================== ======================================================================================================
IEXEC_DATASET_FILENAME      name of the dataset filename that is in the description of task
IEXEC_INPUT_FILES_FOLDER    name of the folder (in the container) where are all the input files and dataset
IEXEC_NB_INPUT_FILES        number of input files described in the task and that have been downloaded to IEXEC_INPUT_FILES_FOLDER
IEXEC_INPUT_FILE_NAME_1     name of the first input file in the list of input files given in parameters of the task.
IEXEC_INPUT_FILE_NAME_2     name of the second input file in the list of input files given in parameters of the task.
IEXEC_INPUT_FILE_NAME_n     name of the nth input file in the list of input files given in parameters of the task.
========================== ======================================================================================================

There will be as many IEXEC_INPUT_FILE_NAME_* variables as there are input files in the parameters of the task.

BoT variables
-------------

Some additional variables are available regarding the Bag Of Task, in order for the worker to know which part of the BoT it is processing:

===================== ==================================================================================
Variables               Meaning
===================== ==================================================================================
IEXEC_BOT_SIZE         Size of the BoT, which means that it is the number of Tasks contained in the BoT.
IEXEC_BOT_FIRST_INDEX  Index of the first task in the BoT.
IEXEC_BOT_TASK_INDEX   Index of the current task that is being processed.
===================== ==================================================================================

