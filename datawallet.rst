===================
Data Wallet
===================

************
Overview
************

An essential feature of the v3 of iExec is the ability to monetize all the aspects of a computation: not only the hardware resources (Workers), but also the code and the data used in the computation (owned respectively by the dApp developer and the Data Owner). In this setting, the iExec software stack must guarantee the security of the code and data, and ensure no one can get access to them if not in the context of a blockchain-verified iExec transaction. Given the decentralized nature of the iExec platform, where computation can be performed by any anonymous user on unknown machines, the only way to fulfill these requirements is to take advantage of the recent developments in hardware-enforced secure execution. Consequently iExec v3 makes use of the Intel SGX technology to create a hardware-protected Trusted Execution Environment (TEE), where the code and data are out of reach and invisible even for a user with physical access to the execution machine.

Challenges/Requirements/Threat model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Data security is an important concern and a challenging problem even in the traditional, private and centralized computing environment. In an open and decentralized computing model like the iExec platform, these challenges are compounded several times. In addition to security against a traditional attack (eg a remote attacker exploiting a software/OS vulnerability to breach into a server and leak data), iExec must solve the following technical challenges:

* The owner of the machine where the computation takes place is anonymous and unvetted. We must assume he may steal any code or data available, especially if they are valuable.
* The data must be accessible publicly: since there can’t be any synchronisation/communication between a data owner and a data user, the (encrypted) dataset must be publicly available, on a third-party server or a decentralized file system like IPFS.
* Access must be granted in an asynchronous way: once again since there is no communication between a data owner and a data recipient/user, access to the data must be granted ahead-of-time by the data owner. In particular this prevents the use of re-encryption schemes, where the data would be encrypted specifically for the recipient using his public key. On the contrary in the iExec setting the symmetric key used for the data set/ application has to be available to the vetted application (ie whose end user has paid for the data, and is running authorized code) without supervision from the data owner.

Our solution is two-fold:

* Firstly, the iExec software supports the execution of tasks inside a trusted execution environment, using Intel SGX enclave technology. The code inside the enclave is guaranteed to behave as intended. It is responsible for retrieving the data and loading the right application to process. The code and the data inside an enclave are completely out-of-reach from code outside of the enclave - even for simple read access.

                                                   .. image:: ./_images/SGX.jpg

* Second we provide an auditable (open source) `Secret Management Service <https://github.com/iExecBlockchainComputing/SMS>`_ (SMS) software, whose purpose is to securely store the keys for the different actors involved (dataset owner, computation beneficiary). This solves the problem of delegating access rights: in essence the user delegates managing access to his data to the SMS. Of course the SMS has the same security challenges as the Worker (we must ensure it runs as expected, doesn’t spill the secrets, and is not spoofed by the owner of the machine on which it runs) hence **it must also run inside an SGX enclave**. As of now there is only one SMS running on iExec's servers, but we intend to open-source the code in the near future, so that anyone will be able to run its own SMS. Data owners will then be able to choose the SMS they trust the most. The address of the chosen SMS is parametrizable in the iExec SDK.

Security guarantees
~~~~~~~~~~~~~~~~~~~~

#. Private code and data will be symmetrically encrypted whenever publicly available (IPFS, public repository). Decryption will only take place inside an attested SGX enclave, whose code has been authorized by the data owner.
#. Private code and data will be encrypted client-side, on the machine of their owner, then pushed on a public data repository. The corresponding keys will only be communicated over TLS channels established with auditable programs, running in attested enclave.
#. The Data owner may limit the use of their data to a set of applications. In this case the key will only be sent to an attested enclave whose MREnclave matches the one whitelisted by the Data Owner.
#. The Data Owner may limit the use of their data to a reduced number of end users (beneficiaries). In this case, the result of the computation will be encrypted by the Worker and only the beneficiary will be able to decrypt it.
#. At all times, keys to the data will only be accessible by:

	* The iExec SMS service (open-source iExec code, attested by Intel IAS)
	* The iExec Dapp (with an MREnclave either guaranteed by its author or by an independent auditor) white-listed by the Data provider.
	* The enclave-based Scone software component, audited by a third-party and attested by Intel IAS.

****************
Implementation
****************

Across the iExec software stack there are two components that are mainly concerned with the security of data: the Worker and the SMS.
The Worker is made of two parts: the In-Enclave (trusted) Worker and the untrusted, standard Worker. All data-sensitive operations (ie handling the dataset keys and decryption, and subsequent processing of the data) are handled by the In-Enclave worker. The untrusted Worker handles interaction with the blockchain and the retribution process (PoCO) but it has no access to the data.
The Secret Management Service is an independent service, hosted either by the data provider or by a third-party, that implements a keystore in a secure, remotely attested enclave. It is mainly there to ensure that the keys are accessible at all times by an authorized user, ie that the computation can take place without an explicit authorization from the data provider.
The Worker and the SMS both run in an SGX enclave; they are managed by an instance of the Palaemon software running in its own enclave. The Palaemont software is responsible for creating the enclave, initializing it with the iExec software and attesting it locally.

Protocol:
~~~~~~~~~~
Here are the steps of an end-to-end encrypted execution:

#. The Data Owner symmetrically encrypts his dataset and pushes the result on an online repository (IPFS). He then assigns a unique (Ethereum) address to this data, by creating a dedicated smart contract for it.
#. The Data Owner establishes a secure connection with the SMS of his choice. The connection is established after verifying the SMS enclave certificate that proves to the Data Owner that the other endpoint of the TLS channel is an SGX enclave running the proper SMS software.
#. The Data Owner then pushes the symmetric key and address of the encrypted data, authenticated (signed) with his Ethereum private address, to the SMS enclave (over TLS). The SMS keeps a mapping {data key ⇔ data address}.
#. After being assigned a computation, the Worker receives the information on the task  which includes which app to run and which dataset to use (by reading them on the blockchain). With the address of the dataset he is able to retrieve which SMS is holding the corresponding symmetric key.
#. The Worker then asks the Palaemon service to create an enclave, load it with the downloaded app and attest it. Palaemon delivers a TLS certificate to the enclave, authenticated with the MREnclave (ie attesting the enclave is running the intended app).
#. The Worker downloads the encrypted data and place it in a special directory available reachable by the enclave Worker.
#. The enclave Worker then requests the data key to the SMS. The request is signed with the enclave private key; the request also contains the enclave certificate delivered by Palaemon (which links the public key corresponding to the enclave private key to the MREnclave of the enclave Worker). When receiving this request, the SMS retrieves the MREnclave vetted by the data owner (by reading it on the blockchain).
#. The SMS then verifies the certificate, and compare the MREnclave of the certificate with the one on the blockchain corresponding to the Dataset address. If the comparison is positive, it sends the data key (over the TLS channel)

Analysis:
~~~~~~~~~~

Since the data is encrypted by the data owner, their confidentiality and integrity depends on two conditions:

#. The keys should not leak during transmission to the worker (machine) where the computation will take place
#. The keys should not leak during the computation.

* 1. is ensured by symmetric encryption of the data, transmitting the keys over TLS, and by the design of the SMS, which transmits the keys only to an enclave with the right MREnclave (and thus the right application), and only for a computation whose beneficiary has been authorized by the Data Owner. By running the SMS inside an enclave and attesting it before sending it the keys, the Data Owner can make sure he is communicating with a proper SMS that will run as intended.
* 2. is ensured by remotely attesting the Worker enclave and auditing the code that runs inside it, making sure the code is not malicious and won’t leak the data or its keys.

************
Tutorial
************

In this tutorial we describe how to realize a fully secured computation with end-to-end data encryption using the iExec stack. As usual with the iExec platform, there are 4 different workflows, depending on your role in the transaction: dApp developer, data provider, hardware resource provider (worker) and computation requester.


Worker
~~~~~~~~

As a worker you need SGX-compatible hardware if you want to perform TEE-based computations. An unofficial list of SGX enabled CPU is available `here <https://github.com/ayeks/SGX-hardware>`_. Basically if your computer was built after 2016 it should be good.
In addition to an SGX-compatible CPU you also need to make sure the BIOS of your machine support the SGX extension. Most mainstream brand of computer (Dell, HP,...) do. If the SGX option is available in your BIOS then you need to enable it.


                                                   .. image:: ./_images/bios1.jpeg

The next step is to install the drivers from Intel for the SGX extension. This can be done in one command line using the following script (on Ubuntu):

.. code-block:: bash

	# curl -fssl https://raw.githubusercontent.com/SconeDocs/SH/master/install_sgx_driver.sh | bash

That’s it! Now you can register at your scheduler as an SGX compatible worker, and you’ll soon receive requests for SGX jobs.


Data provider
~~~~~~~~~~~~~~~~~~

If you want to protect your dataset you need to encrypt it before making it available on the iExec platform. There are two ways to encrypt your dataset, and only one of them is SGX compatible: see the `SDK tutorial <https://github.com/iExecBlockchainComputing/iexec-sdk/>`_ for more info.
First, initialize the folder structure:

.. code-block:: bash

        $ iexec tee init

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

        $ cp /path/to/your/dataset tee/original-dataset/dataset-name
        $ iexec tee encrypt-dataset  --algorithm scone

This command will encrypt your dataset to enable its use in a scone runtime execution. It will also write the corresponding key and tag in a :code:`.tee-secrets/dataset/<dataset-name>.scone.secret` file.

**You need to upload the encrypted dataset to a public server (for example a Github repository or on IPFS).**

Create the dataset contract template:

.. code-block:: bash

	$ iexec dataset init

This will create the template for the dataset info in the *iexec.json*. **Don't forget to add the multiaddress of your dataset (where the dataset will be available to download)**. The checksum field is not used currently, and can be left blank.

.. code-block:: bash

        "dataset": {
          "owner": "0x9A07Ea49a32C1E69eD7B6dFe1aa1C19181465C52",
          "name": "test_sgx",
          "multiaddr": "https://raw.githubusercontent.com/iExecBlockchainComputing/test_sgx/master/sgx_data.zip",
          "checksum": "0x0000000000000000000000000000000000000000000000000000000000000000"
        }

Then deploy your dataset:

.. code-block:: bash

	$ iexec dataset deploy

.. code-block:: bash

        ~/SGX/test_sgx/test$ iexec dataset deploy
        ℹ iExec SDK update available 3.0.33 →  3.0.34, Run "npm -g i iexec" to update

        ℹ using chain [kovan]
        ? Using wallet UTC--2019-05-28T16-00-29.164000000Z--9A07Ea49a32C1E69eD7B6dFe1aa1
        C19181465C52
        Please enter your password to unlock your wallet [hidden]
        ✔ Deployed new dataset at address 0x0bF2AEb5e7FCE90DCb39FEEaC49Ce44893CAd31d

Once you dataset is deployed you can push its secret (encryption key and hash of the data) to the SMS. This is done simply with the SDK:

.. code-block:: bash

	$ iexec tee push-secret --dataset <Dataset contract address> --secret-path <$PWD/.tee-secrets/dataset/<dataset-name>.scone.secret>


Initialize your dataset order:

.. code-block:: bash

	$ iexec order init

...and edit your dataset order in the *order.json* file, to copy-paste the address of your dataset. Here you can set the price of your dataset, and the number of utilisations. You can also whitelist \
the app and worker pool that will be allowed to use your dataset. **Don't forget to replace the tag, from 0x00..000 to 0x00...001 (as seen below).**

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

Once your order is ready you can sign it, and send it to the potential user of your dataset. You can also publish it on the iExec marketplace with the SDK:

.. code-block:: bash

        $ iexec order sign --dataset
	$ iexec order publish --dataset


DApp developer
~~~~~~~~~~~~~~~


Background
************

At its core the Intel SGX technology relies on the creation of special zones in memory called enclaves. Access to this zone is protected by the CPU, so that only code from inside the zone can access data in the enclave. If a code from outside the enclave - whatever its privilege level, even OS or hypervisor code -  tries to read a memory location that is part of the enclave the CPU will return an error.
The drawback is that whenever your program needs to use code outside the enclave - for example OS code  (eg system calls) for network or file system access - it needs to perform a special sequence of CPU instruction to leave the enclave securely. As a result to run a program natively you would need to rewrite it using Intel SDK and call these instructions manually, an impractical and potentially complex task.
To avoid this and make the use of SGX through iExec as developer friendly as possible, iExec provides a transparent integration with Scone, a runtime component developed by Scontain that allows to run applications in SGX enclaves in an unmodified way. We provide several docker images, that already include the Scone components as well as iExec integration code, that make the development of iExec-ready, SGX-enabled dApp as simple as a few Dockerfile lines.

Example: creating a Python 3 SGX dApp
**************************************

Here we explain how to create an SGX enabled python app. We provide a Github repository with several examples. Our SGX framework is based on the Scone runtime, that allows us to run unmodified apps inside SGX enclaves.
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

        $ docker build --no-cache -t iexechub/myapp:latest .

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
        Successfully tagged iexechub/sgx-app:latest

You should copy this value and paste it in the iexec.json file, as the app "mrenclave" value:

.. code-block:: bash

        "app": {
          "owner": "0x9A07Ea49a32C1E69eD7B6dFe1aa1C19181465C52",
          "name": "test_sgx",
          "type": "DOCKER",
          "multiaddr": "iexechub/myapp:latest",
          "checksum": "0xc4f18d6e024ac1bd1b0cf08484ca7baaf4c63eb67a20fefe51017424df2a5179",
          "mrenclave": "3b62fef269341bc93238580b516d2d934e1264e7442e484d4d459a9abc519a76|b873e72c7687e95d734b7905e07c51d8|b84bc68bae8cdc8703ca4525b2cc16deffe9def4247498ebcc467830a67caf6d"
        },

**Don't forget to also modify the "multiaddr" field, so that it points towards you app image once you've pushed it on a Docker repository.** You can then deploy you app, following the normal iExec workflow:

.. code-block:: bash

        $ docker push iexechub/myapp:latest

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

Edit your app order, by copy-pasting your dApp contract address (in our example 0x6E519c9887cD2d59918e4EF049b5d9fF489E6E2f), and setting the price and number of use of your dApp (and potentially restriction on dataset, worker and requester allowed to use you dApp).
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

Once your order is ready you can sign it, and send it to the potential user of your dApp. You can also publish it on the iExec marketplace with the SDK.

.. code-block:: bash

        $ iexec order sign --app
        $ iexec order publish --app


Computation requester/ beneficiary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As a computation requester it is your choice to decide whether or not your execution should use iExec Data wallet.

**Step 1: Create and push your encryption key**

One of the most interesting features of iExec Data wallet is the possibility to ask for your result to be encrypted inside the TEE: that is, only you will be able to read them. To allow this you need to generate a PKC key pair, and upload the public part to the SMS. This can be done in just one step with the iExec SDK:

.. code-block:: bash

	$ iexec tee generate-beneficiary-keys

Then you can push your public key to the SMS:

.. code-block:: bash

	$ iexec tee push-secret


**Step 2: Order a E2E encrypted computation on iExec**

You can then follow the normal workflow to buy a computation as described in the `doc for the normal workflow <https://docs.iex.ec/requester.html>`_

At this point you may use either the SDK or the web interface available at market.iex.ec.

**Option A: using the SDK**

.. code-block:: bash

	$ iexec order init

As in the normal iExec workflow, you should fill all the info needed in the iexec.json file (app, dataset, price, category). In the case of an SGX execution there are however two differences:

#. You should replace the *tag* by 0x0...01 (instead of 0x00...000)
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

**Option B: using the web interface**

You can also use the iExec marketplace's web interface. Likewise, you need to fill the address of the dataset and app you want to use. Don't forget to check the "TEE" checkbox.

                                                .. image:: ./_images/BuyComputation.png


**Step 3: Download and decrypt your results**

Once the computation is finished you can download the result using the iExec marketplace web interface. You can then decrypt your result with the SDK:

.. code-block:: bash

	$ iexec tee decrypt-results <encryptedResultsFilePath>

And that's it! Your computation was executed in a protected enclave, and encrypted in-place: no one on Earth except you will be able to read the results.
