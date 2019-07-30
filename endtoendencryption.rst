==============================
TEE End-to-end encryption (Intel SGX)
==============================

The following tutorial will show how you can execute a fully secured computation with end-to-end data encryption in a Trusted Execution Environment (TEE) using the iExec stack.

Preparing the application, dataset and the computation request for end-to-end TEE encryption will require some extra steps - but they are all embedded into the iExec SDK.

The TEE computation must run on a TEE compatible worker. The requester will therefore need to select a SGX hardware enclave compatible workerpool.

You can find all the available TEE computing resources by selecting the 'TEE' checkbox on the iExec Marketplace UI.

.. image:: ./_images/teecheckbox.png


Encrypt and register the dataset
--------------------------------

If you want to protect your dataset you will need to encrypt it, before making it available on the iExec platform. This is quite simple to do using the iExec SDK:
see the `SDK tutorial <https://github.com/iExecBlockchainComputing/iexec-sdk/>`_ for more info.

First, initialize the folder structure

.. code-block:: bash

       iexec tee init
   ✔ created TEE folders tree structure

This command will create the following folders:

.. code-block:: bash

        ├── tee
        │   ├── encrypted-dataset
        │   └── original-dataset
        └── .tee-secrets
            ├── beneficiary
            └── dataset

Copy your dataset in the *original-dataset/*  folder, then encrypt it with the SDK:

.. code-block:: bash

        cp /path/to/your/dataset tee/original-dataset/dataset-name
        iexec tee encrypt-dataset  --algorithm scone

This command will encrypt your dataset to enable its use in a scone runtime execution.

It will also write the corresponding key and tag in a :code:`.tee-secrets/dataset/<dataset-name>.scone.secret` file.

**You need to upload the encrypted dataset to a public server (for example a Github repository or on IPFS).**

Check with wget command or your web browser to check the encrypted dataset has a public access.


Now, you can deploy the dataset on the iExec Marketplace

Set up a configuration file.

.. code-block:: bash

    iexec dataset init


This will create the template for the dataset info in the *iexec.json*.

Then edit the iexec.json to describe your application.

**Don't forget to add the multiaddress of your dataset (where the dataset will be available to download)**.

The checksum field is not used currently, and can be left blank.

.. code-block:: bash

        "dataset": {
          "owner": "0x9A07Ea49a32C1E69eD7B6dFe1aa1C19181465C52",
          "name": "test_sgx",
          "multiaddr": "https://raw.githubusercontent.com/iExecBlockchainComputing/test_sgx/master/sgx_data.zip",
          "checksum": "0x0000000000000000000000000000000000000000000000000000000000000000"
        }

Then deploy your dataset.

.. code-block:: bash

        iexec dataset deploy
        ℹ iExec SDK update available 3.0.33 →  3.0.34, Run "npm -g i iexec" to update

        ℹ using chain [kovan]
        ? Using wallet UTC--2019-05-28T16-00-29.164000000Z--9A07Ea49a32C1E69eD7B6dFe1aa1
        C19181465C52
        Please enter your password to unlock your wallet [hidden]
        ✔ Deployed new dataset at address 0x0bF2AEb5e7FCE90DCb39FEEaC49Ce44893CAd31d


Once your dataset is deployed, you can push its secret (encryption key and hash of the data) to the SMS.
This is done simply with the SDK:

.. code-block:: bash

       iexec tee push-secret --dataset <Dataset contract address> --secret-path <$PWD/.tee-secrets/dataset/<dataset-name>.scone.secret>


Publish the dataset order

Create an order template

.. code-block:: bash

	iexec order init --dataset

...and edit your dataset order in the *order.json* file,

Edit the order part in iexec.json to describe the dataset.

===================== ==========================================================
Parameter               Meaning
===================== ==========================================================
 dataset                dataset address
 datasetprice           dataset price
 volume                 number of order created
 tag                    tag for extra computational requirement (*)
 datasetrestrict:       restricted to dataset (*)
 workerpoolrestrict     restricted to workerpool (*)
 requesterrestrict:     restricted to requester (*)
===================== ==========================================================

**The dataset has to be enabled with the corresponding tag 0x0000000000000000000000000000000000000001**


.. code-block:: bash

        "datasetorder": {
          "dataset": "0x0bF2AEb5e7FCE90DCb39FEEaC49Ce44893CAd31d",
          "datasetprice": 1000,
          "volume": 1000000,
          "tag": "0x0000000000000000000000000000000000000000000000000000000000000001",
          "apprestrict": "0x0000000000000000000000000000000000000000",
          "workerpoolrestrict": "0x0000000000000000000000000000000000000000",
          "requesterrestrict": "0x0000000000000000000000000000000000000000"
        }

Once your order is ready, you can sign it before sending it to the potential user of your dataset. You can also publish it on the iExec marketplace using the SDK:

.. code-block:: bash

        $ iexec order sign --dataset
	$ iexec order publish --dataset



Build an SGX-enabled application
----------------------------------

Background: porting an application to SGX
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At its core, Intel SGX technology relies on the creation of special zones in memory called enclaves. Access to this zone is protected by the CPU, so that only code from inside the zone can access data in the enclave. If a code from outside the enclave - whatever its privilege level, even OS or hypervisor code -  tries to read a memory location that is part of the enclave, the CPU will return an error.
The drawback is that whenever your program needs to use code outside the enclave - for example OS code  (eg system calls) for network or file system access - it needs to perform a special sequence of CPU instruction to leave the enclave securely. As a result to run a program natively you would need to rewrite it using Intel SDK and call these instructions manually, an impractical and potentially complex task.
To avoid this and make the use of SGX through iExec as developer friendly as possible, iExec provides a transparent integration with Scone, a runtime component developed by Scontain that allows to run applications in SGX enclaves in an unmodified way. We provide several docker images, that already include the Scone components as well as iExec integration code, that make the development of iExec-ready, SGX-enabled dApp as simple as a few Dockerfile lines.

Example: creating a Python 3 SGX dApp
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here we explain how to create an SGX enabled python app.

Our SGX framework is based on the Scone runtime, that allows us to run unmodified apps inside SGX enclaves.

Hence your Docker image should be built from our python_sgx image available on our docker repository.

We provide a `Github repository <https://github.com/iExecBlockchainComputing/sgx-apps>`_ with several examples, that show how to build an SGX-enabled docker image.

**Step 1: Create your app folder**

First clone the test-sgx repo and create the directories for your app, and initialize the iExec SDK:

.. code-block:: bash

        git clone https://github.com/iExecBlockchainComputing/sgx-apps.git
        cd test-sgx
        mkdir my-app
        mkdir my-app/src
        cd my-app
        iexec init
        iexec app init

After this you should copy the file for your app (python scripts) in the my-app/src folder.

**Step 2: Copy and edit the Dockerfile**

.. code-block:: bash

        cp ../Dockerfile Dockerfile
        nano  Dockerfile

The Dockerfile provides template for a generic python app. **You should edit it to add the libraries and packages your app may need, as well as its specific Docker entrypoint**.


**Step 3: Build the Docker image and copy the fingerprint**

Build your docker image in the normal way. You may need the no-cache option if you've already build it once (otherwise it won't print the MREnclave).

.. code-block:: bash

        $ docker build --no-cache -t iexechub/myapp:1.0.0 .

The build might take some time. At the end of the build process, the docker script will display the "mrenclave" value, as shown below:

.. code-block:: bash

        Added region /signer to file system protection file fspf.pb new AES-GCM tag: b074e8d611711a809e09ae48b26a2244
        Added files to file system protection file fspf.pb new AES-GCM tag: 1798fa5a4f1311e51b2ac1435f1c6a38
        Added region /app to file system protection file fspf.pb new AES-GCM tag: 43e2f518d28890425fb8f6f20acb2856
        Added files to file system protection file fspf.pb new AES-GCM tag: ca4a9cdf07bc74ed535480a8562280f6

        ########################################################
        MREnclave: 3b62fef269341bc93238580b516d2d934e1264e7442e484d4d459a9abc519a76|b873e72c7687e95d734b7905e07c51d8|b84bc68bae8cdc8703ca4525b2cc16deffe9def4247498ebcc467830a67caf6d
        ########################################################

        Removing intermediate container 6994b08919a1
         ---> 2ba28d9ea4c2
        Step 7/7 : ENTRYPOINT python3 /app/app.py
         ---> Running in 5ca393ffd291
        Removing intermediate container 5ca393ffd291
         ---> 7144abe35d7b
        Successfully built 7144abe35d7b
        Successfully tagged iexechub/sgx-app:1.0.0 

You should copy this value and paste it in the iexec.json file, as the app "mrenclave" value:

.. code-block:: bash

        "app": {
          "owner": "0x9A07Ea49a32C1E69eD7B6dFe1aa1C19181465C52",
          "name": "test_sgx",
          "type": "DOCKER",
          "multiaddr": "iexechub/myapp:1.0.0",
          "checksum": "0xc4f18d6e024ac1bd1b0cf08484ca7baaf4c63eb67a20fefe51017424df2a5179",
          "mrenclave": "3b62fef269341bc93238580b516d2d934e1264e7442e484d4d459a9abc519a76|b873e72c7687e95d734b7905e07c51d8|b84bc68bae8cdc8703ca4525b2cc16deffe9def4247498ebcc467830a67caf6d"
        },

**Don't forget to also modify the "multiaddr" field, so that it points towards you app image once you've pushed it on a Docker repository.** You can then deploy you app, following the normal iExec workflow:

.. code-block:: bash

        $ docker push iexechub/myapp:1.0.0

.. code-block:: bash

        $ iexec app deploy

Once your app is deployed the SDK will display the Ethereum address of your app contract.

.. code-block:: bash

        ℹ iExec SDK update available 3.0.33 →  3.0.34, Run "npm -g i iexec" to update
        ℹ using chain [kovan]
        ? Using wallet UTC--2019-05-28T16-00-29.164000000Z--9A07Ea49a32C1E69eD7B6dFe1aa1
        C19181465C52
        Please enter your password to unlock your wallet [hidden]
        ✔ Deployed new app at address 0x6E519c9887cD2d59918e4EF049b5d9fF489E6E2f

You need to copy this address and paste it in your app order available in the order.json file:

.. code-block:: bash

        $ iexec order init
        $ nano iexec.json

Edit your app order, by copy-pasting your dApp contract address (in our example 0x6E519c9887cD2d59918e4EF049b5d9fF489E6E2f), and setting the price and number of use of your dApp (and potentially restrictions on dataset, worker and requester allowed to use your dApp).
**Don't forget to replace the tag, from 0x00..000 to 0x00...001 (as seen below).**

.. code-block:: bash

        "apporder": {
          "app": "0x6E519c9887cD2d59918e4EF049b5d9fF489E6E2f",
          "appprice": 10000,
          "volume": 1000000,
          "tag": "0x0000000000000000000000000000000000000000000000000000000000000001",
          "datasetrestrict": "0x0000000000000000000000000000000000000000",
          "workerpoolrestrict": "0x0000000000000000000000000000000000000000",
          "requesterrestrict": "0x0000000000000000000000000000000000000000"
        }

Once your order is ready, you can sign it, and send it to the potential user of your dApp. You can also publish it on the iExec marketplace with the SDK.

.. code-block:: bash

        $ iexec order sign --app
        $ iexec order publish --app

That’s it! Your app is now SGX compatible. Now you can deploy it using iExec SDK, following the normal dApp workflow (see `tutorial <https://docs.iex.ec/appprovider.html#deploy-your-dapp>`_).



Request a computation
---------------------

As a computation requester it is your choice to decide whether or not your execution should use iExec Data wallet.

**Step 1: Create and push your encryption key**

One of the most interesting features of iExec Data wallet is the possibility to ask for your result to be encrypted inside the TEE: meaning only you will be able to read them. To allow for this you will need to generate a PKC key pair, and upload the public part to the SMS. This can be done in just one step with the iExec SDK:

.. code-block:: bash

	iexec tee generate-beneficiary-keys

Then you can push your public key to the SMS:

.. code-block:: bash

	$ iexec tee push-secret


**Step 2: Order a E2E encrypted computation on iExec**


You can then follow the normal workflow to buy a computation as described in the `doc for the normal workflow <https://docs.iex.ec/requester.html>`_

At this point you may use either the SDK or the web interface available at `market.iex.ec <https://market.iex.ec>`_.

**Option A: using the SDK**

You can then follow the normal workflow to buy a computation as described in the `tutorial <https://docs.iex.ec/appprovider.html#deploy-your-dapp>`_

.. code-block:: bash

	$ iexec order init

As in the normal iExec workflow, you should fill all the info needed in the iexec.json file (app, dataset, price, category).

In the case of an SGX execution there are however two differences:

#. You must set the *tag* 0x0...01 (instead of 0x00...000)
#. In the *params* field you should put the command to launch your app

.. code-block:: bash

	"requestorder": {
	"app": "0xAAdC3C643b79dbf8b761bA62283fF105930B20eb",
	"appmaxprice": 1500,
	"dataset": "0x570280a48EA01a466ea5a88d0f1C16C124BCDc3E",
	"datasetmaxprice": 12000,
	"workerpool": "0x0000000000000000000000000000000000000000",
	"workerpoolmaxprice": 5000,
	"volume": 1,
	"category": 3,
	"trust": 5,
	"tag": "0x0000000000000000000000000000000000000000000000000000000000000001",
	"beneficiary": "0xC08C3def622Af1476f2Db0E3CC8CcaeAd07BE3bB",
	"callback": "0x0000000000000000000000000000000000000000",
	"params": "python app/app.py",
	"requester": "0xC08C3def622Af1476f2Db0E3CC8CcaeAd07BE3bB"
	}

Then sign your orders, and publish your request order:

.. code-block:: bash

	$ iexec order sign
	$ iexec order publish --request

If your order is matched with the required components (app, dataset, worker), the computation will happen automatically, in a totally secure way.

**Option B: Using the web interface**

You can also use the iExec marketplace's web interface. Likewise, you need to fill the address of the dataset and app you want to use. Don't forget to check the "TEE" checkbox.

                                                .. image:: ./_images/BuyComputation.png


**Step 3: Download and decrypt your results**

Once the computation is finished you can download the result using the iExec marketplace web interface. You can then decrypt your result with the SDK:

.. code-block:: bash

	$ iexec tee decrypt-results <encryptedResultsFilePath>

And that's it! Your computation was executed in a protected enclave, and encrypted in-place: no one on Earth except you will be able to read the results.
