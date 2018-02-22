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

.. note:: Please check that application in your docker container executes successfully before submitting it to iExec.

.. note:: All `dirinuri` files are avaliable in `/host` directory.

.. warning:: The working directory of container is forced to `/host` within execution.

.. warning:: Don't put your application or any data to `/host` directory cause this directory will mounted as a docker volume.
             So the your data will not be available.

Push your image to Dockerhub
----------------------------
You must push your image to a public repository at Docker Hub.
Before execution iExec worker will pull image from public repository.

Get Dapp Sample
---------------
The easiest way to start building your dapp at iExec is to get dapp example provided by our team.

Get ffmpeg Dapp:

.. code:: bash

   git clone https://github.com/iExecBlockchainComputing/iexec-dapp-samples.git --branch ffmpeg


Modify Dapp Sample
------------------
We must modify two files in order to adapt Dapp Sample to your application.


Modify ./contracts/Ffmpeg.sol file
**********************************

.. code:: javascript

    pragma solidity ^0.4.11;
    import "iexec-oracle-contract/contracts/IexecOracleAPI.sol";
    contract Ffmpeg is IexecOracleAPI{

        uint public constant DAPP_PRICE = 0;
        string public constant DAPP_NAME = "Ffmpeg";

        function Ffmpeg (address _iexecOracleAddress) IexecOracleAPI(_iexecOracleAddress,DAPP_PRICE,DAPP_NAME){

        }

    }

* Modify contract name
* Modify DAPP_PRICE
* Modify DAPP_NAME

.. warning:: Don't forget to rename Ffmpeg.sol file

Modify ./iexec.js file
**********************

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

* `name` - application name
* `app.type` - type of application
* `work.cmdline` - command which will be executed in your container
* `work.dirinuri` - file that will be downloaded to containers `/host` directory (Can be used for source data)

  * Can be a single file
  * Can be `.zip` archive, which will be decompressed automatically

Deploy your Dapp
----------------

Now your dapp is ready. So you can deploy it with iExec SDK.
