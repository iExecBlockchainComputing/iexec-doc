How to privately execute a task
===============================


iExec provides a full end to end data protection for blockchain-based computation using Intel® SGX.

End-to-end protection means full protection of the application data, user data, embedded data as well as application output data.

Intel® Software Guard Extensions (SGX) is a technology that runs code and data in CPU-hardened “enclaves” or a ‘Trusted Execution Environment’ (TEE).

The enclave is a trusted area of memory where critical aspects of the application functionality are protected, helping keep code and data confidential and unmodified.

This following section outlines step by step how to run an application protected by Intel® SGX.

The solution is available through the iExec SDK.

Head to the `Getting Started`_ section to install and use the iExec SDK for task execution.

Your account must be topped up with RLC in order to trigger the execution of an application.

.. _Getting Started: /sdk.html

As an illustration, let's create an image with the 3D rendering software `Blender <https://www.blender.org/>`_, while preserving the privacy of the your input data.


1. Encrypt and push data on a public server

Let’s locally encrypt the data and push it on a public file hosting service, so that the worker is able to access it:

.. code-block:: bash

    iexec tee init # create iExec trusted execution folders tree


Download the blender model `iexec-rlc.blend <https://raw.githubusercontent.com/iExecBlockchainComputing/iexec-dapps-registry/master/iExecBlockchainComputing/Blender/iexec-rlc.blend>`_,
and copy the file in the *./tee/inputs* folder.

.. code-block:: bash

    cp iexec-rlc.blend ./tee/inputs

Encrypt the input data:

.. code-block:: bash

    iexec tee encryptedpush --application 0x2f3422f2805693cf741ee32707d57923ef6fa55f
    ℹ using chain [kovan]
    ⠅⡘ ▶ encrypting data from /home/eric/pm/test/tee/inputs and uploadingcli: Pulling from iexechub/sgx-scone
    Digest: sha256:e5f5bd685c211c58f2d6429dc21e5f61b27ca04b0d1fba8d0898fa30f0b36800

    Status: Image is up to date for iexechub/sgx-scone:cli

    ⡃⠀ ▶ encrypting data from /home/eric/pm/test/tee/inputs and uploading
    "cmdline": --sessionID 6860291353034118628213787713/application --secretManagementService 87.190.236.136  --url https://transfer.sh/NAsUs/scone-upload.zip

    ✔ data encrypted and uploaded

The above-mentioned command will return the command line parameters in string format that will be used in the next step.


2. Trigger trusted application execution


Prepare a work order and trigger the trusted application execution:

.. code-block:: bash

    iexec order init --buy # init work order fields in iexec.json


Now open the iexec.json config file, and edit the app and command line fields:

  - Address for the blender app: 0x2f3422f2805693cf741ee32707d57923ef6fa55f
  - Command line provided by the " iexec tee encryptedpush ..." command

.. code-block:: javascript

     "order": {
        "buy": {
          "app": "0x2f3422f2805693cf741ee32707d57923ef6fa55f",
          "dataset": "0x0000000000000000000000000000000000000000",
          "params": {
          "cmdline": "--sessionID 6860291353034118628213787713/application --secretManagementService 87.190.236.136  --url https://transfer.sh/NAsUs/scone-upload.zip"
          }
        }
      }
    }

Select a worker pool supporting SGX:

.. code-block:: bash

    iexec orderbook show --category 5

Select an order from the worker pool with the address 0x49327538C2f418743E70Ca3495888a62B587A641.
This worker pool supports SGX.

Fill the selected order:

.. code-block:: bash

    iexec order fill 1963
    ℹ using chain [kovan]
    ℹ app price: 1 nRLC for app 0x2f3422f2805693cf741ee32707d57923ef6fa55f
    ℹ workerpool price: 15863 nRLC for workerpool 0x49327538c2f418743e70ca3495888a62b587a641
    ℹ work parameters:
    cmdline: --sessionID 6860291353034118628213787713/application --secretManagementService 87.190.236.136  --url https://transfer.sh/NAsUs/scone-upload.zip

    ? Do you want to spend 15864 nRLC to fill order with ID 1963 and submit your work Yes
    ✔ Filled order with ID 1963
    ✔ New work at 0x6bd60b2c01a161c46915c6a12553eaaee332f785 submitted to workerpool 0x49327538c2f418743e70ca3495888a62b587a641

Monitor your order:

.. code-block:: bash

    iexec work show 0x6bd60b2c01a161c46915c6a12553eaaee332f785
    ℹ using chain [kovan]
    ✔ work 0x6bd60b2c01a161c46915c6a12553eaaee332f785 status is COMPLETED, details:
    m_workerpool:          0x49327538c2f418743e70ca3495888a62b587a641
    m_params:              {"cmdline":"--sessionID 6860291353034118628213787713/application --secretManagementService 87.190.236.136  --url https://transfer.sh/NAsUs/scone-upload.zip"}
    m_requester:           0x7800885445a481315fac90a8e8bdb62a0e538b71
    m_app:                 0x2f3422f2805693cf741ee32707d57923ef6fa55f
    m_dataset:             0x0000000000000000000000000000000000000000
    m_emitcost:            1
    m_uri:                 xw://api-tee-pool.iex.ec/7f667265-acb6-40a9-b199-42c07ad40d49
    m_stdout:
    m_resultCallbackProof: 0x9d6f1e3a18cf8bb9d5e8b755abace2d90f26d2196f1df7ed03a2a293d3ec7b7b
    m_iexecHubAddress:     0x12b92a17b1ca4bb10b861386446b8b2716e58c9b
    m_callback:            0x0000000000000000000000000000000000000000
    m_status:              4
    m_marketorderIdx:      1963
    m_stderr:
    m_beneficiary:         0x0000000000000000000000000000000000000000
    m_statusName:          COMPLETED

Download the work result once it is completed:

.. code-block:: bash

    iexec work show 0x6bd60b2c01a161c46915c6a12553eaaee332f785 --download encryptedOutputFiles.zip
    ℹ using chain [kovan]
    ✔ work 0x6bd60b2c01a161c46915c6a12553eaaee332f785 status is COMPLETED, details:
    m_workerpool:          0x49327538c2f418743e70ca3495888a62b587a641
    m_params:              {"cmdline":"--sessionID 6860291353034118628213787713/application --secretManagementService 87.190.236.136  --url https://transfer.sh/NAsUs/scone-upload.zip"}
    m_requester:           0x7800885445a481315fac90a8e8bdb62a0e538b71
    m_app:                 0x2f3422f2805693cf741ee32707d57923ef6fa55f
    m_dataset:             0x0000000000000000000000000000000000000000
    m_emitcost:            1
    m_uri:                 xw://api-tee-pool.iex.ec/7f667265-acb6-40a9-b199-42c07ad40d49
    m_stdout:
    m_resultCallbackProof: 0x9d6f1e3a18cf8bb9d5e8b755abace2d90f26d2196f1df7ed03a2a293d3ec7b7b
    m_iexecHubAddress:     0x12b92a17b1ca4bb10b861386446b8b2716e58c9b
    m_callback:            0x0000000000000000000000000000000000000000
    m_status:              4
    m_marketorderIdx:      1963
    m_stderr:
    m_beneficiary:         0x0000000000000000000000000000000000000000
    m_statusName:          COMPLETED

    ✔ downloaded work result to file /home/eric/pm/test/encryptedOutputFiles.zip.none


Move the result in the './tee/encryptedOutputs/' folder to decrypt the result:

.. code-block:: bash

   mv encryptedOutputFiles.zip.none ./tee/encryptedOutputs/encryptedOutputFiles.zip


Please note that the user who triggered the task (i.e. the SGX application) is the only one able to download the encrypted results.

When the application is triggered in a remote Intel® SGX decentralized node, the application will automatically

  1. Pull the encrypted user input data from remote file system (i.e. pushed in step 2)

  2. Retrieve the secret key (based on the Session ID) from the secret management server via a secure Intel® SGX provision channel

  3. The secret is then used to decrypt the user input data

  4. The decrypted data is used to feed the application execution

  5. The application result is encrypted by the secret key, and the encrypted result is further signed by a secure private key for an attestation of the trusted execution. The signature is verified on the blockchain


The procedure is done automatically in the trusted execution environment
(i.e. Intel® SGX enclave) without any user intervention.

3. Decrypt your result

The last step is decrypting the result:

.. code-block:: bash

    iexec tee decrypt
    ⠉⠙ ▶ decryptingcli: Pulling from iexechub/sgx-scone

    Digest: sha256:e5f5bd685c211c58f2d6429dc21e5f61b27ca04b0d1fba8d0898fa30f0b36800
    Status: Image is up to date for iexechub/sgx-scone:cli

    ⠄⡙ ▶ decryptingArchive:  /encryptedOutputFiles.zip
      inflating: encryptedOutputs/0001.png

      inflating: encryptedOutputs/volume.fspf


    copy file /encryptedOutputs/0001.png to /decryptedOutputs/0001.png

    ✔ data decrypted in folder /home/eric/pm/test/tee/outputs


That’s it! Your completed and secure result is now available in the *./tee/outputs* folder.

Please note that only the corresponding user owns the key to decrypt the application's output result.

.. include:: contactus.rst
