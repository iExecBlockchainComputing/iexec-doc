Docker Application
==================
In this section we will show you how you can create a docker Dapp at iExec.

.. note:: We are considering that you are familiar with docker ecosystem.


Prerequisites
-------------
* Docker CE or EE
* Github
* iExec SDK

Build your image
----------------
Firstly you need to build a docker image which contains your application.

Your image will be launched by iExec worker using following command:

.. code:: bash

   docker run -v ${DIRINURI}:/host -w /host ${XWDOCKERIMAGE} ${CMDLINE}

* `DIRINURI` - directory where `dirinuri` files are accessible
* `XWDOCKERIMAGE` - docker image to run
* `CMDLINE` - command to execute

.. note:: Please check that application in your docker container executes successfully before submitting it to iExec.

.. warning:: The working directory of container is forced to `/host` within execution.

Push your image to Dockerhub
----------------------------
You must push your image to a public repository at Docker Hub.
Before execution iExec worker will pull image from public repository.

Get Dapp Sample
---------------
The easiest way to start building your dapp at iExec is to try our katacoda tutorial:
https://www.katacoda.com/sulliwane/scenarios/ffmpeg

Dapp Files
----------
In katacoda tutorial we are modifing two files in order to adapt Dapp Sample to your application.
In this section we will highlight the most important parts of modified files.

Modifing ./contracts/Ffmpeg.sol file
************************************

.. code:: javascript

    pragma solidity ^0.4.11;
    import "iexec-oracle-contract/contracts/IexecOracleAPI.sol";
    contract Ffmpeg is IexecOracleAPI{

        uint public constant DAPP_PRICE = 0;
        string public constant DAPP_NAME = "Ffmpeg";

        function Ffmpeg (address _iexecOracleAddress) IexecOracleAPI(_iexecOracleAddress,DAPP_PRICE,DAPP_NAME){

        }

    }

* `contract Ffmpeg` - name of the contract. Should be the same as the contract file name. 
* `DAPP_PRICE` - price for dapp execution
* `DAPP_NAME` - dapp name

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
