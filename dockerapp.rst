Docker Application
==================
In this section we will show you how you can create a docker Dapp at iExec.

Build your image
----------------
Firstly you need to build a docker image which contains your application.

Your image will be launched by iExec worker using following command:

.. code:: bash

   docker run -v ${DIRINURI}:/host -w /host ${XWDOCKERIMAGE} ${CMDLINE}

* `DIRINURI` - directory where `dirinuri` files are accessible
* `XWDOCKERIMAGE` - docker image to run
* `CMDLINE` - command to execute

Push your image to Dockerhub
----------------------------
You must push your image to a public repository at Docker Hub.
Before execution iExec worker will pull image from public repository.

Interactive Tutorial
--------------------
The easiest way to start building your dapp at iExec is to try our katacoda tutorial:
https://www.katacoda.com/sulliwane/scenarios/ffmpeg

Dapp Files
----------
In katacoda tutorial we are modifing some files in order to adapt Dapp Sample to your application.

Modifing ./iexec.js file
************************

.. code:: javascript

    module.exports = {
        name: 'Ffmpeg',
        app: {
          type: 'DOCKER',
          envvars: 'XWDOCKERIMAGE=jrottenberg/ffmpeg:scratch',
        },
        work: {
            cmdline:'-i small.mp4 small.avi',
            dirinuri:'http://techslides.com/demos/sample-videos/small.mp4',
        }
    };

* `name` - dapp name
* `app.type` - type of dapp
* `app.envvars` - environment variables passed to your dapp
  
.. warning:: It's very important to set XWDOCKERIMAGE variable. This variable sets the docker image of your dapp. 

* `work.cmdline` - command that will be executed in your container
* `work.dirinuri` - file that will be downloaded to `/host` directory in docker container

  * Can be a single file
  * Can be `.zip` archive, which will be decompressed automatically

Deploy your Dapp
----------------

Now your dapp is ready. So you can deploy it with iExec SDK.
