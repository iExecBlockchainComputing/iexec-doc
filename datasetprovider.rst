Dataset provider
================

| In this section we will show you how you can propose a dataset or any valuable data over iExec infrastructure.
| In the **task-as-a-service** model, each time a task is launched through the iExec network,
| The dataset providers set the price of their datasets. Requesters pay on a pay-per-task basis.
| And you can then withdraw your funds at anytime to your own wallet.


Whitelisting and ordering Dataset owner will manage:

 * who can process the dataset
 * which application can run the dataset
 * make restriction for computing resources
 * set up a cutting-edge pricing management

Deploy your dataset
-------------------

Zip your dataset, or model.

Put the data on public data storage, the dataset must be accessible in direct download.

Set up a configuration file.

.. code-block:: bash

    iexec init --skip-wallet
    iexec dataset init


to set up the template in iexec.json

Then edit the iexec.json to describe your application: name, source, price,...

.. code:: javascript

   "dataset": {
    "owner": "0x9CdDC59c3782828724f55DD4AB4920d98aA88418",
    "name": "Neurovault_brainstatsmaps",
    "multiaddr": "https://raw.githubusercontent.com/ericr6/nilearn/master/nilearn_data.zip",
    "checksum": "0x0000000000000000000000000000000000000000000000000000000000000000"
  },


Then you deploy your dataset:

.. code-block:: bash

    iexec dataset deploy --wallet-file data_owner_wallet
    ℹ using chain [kovan]
    ? Using wallet data_owner_wallet
    Please enter your password to unlock your wallet [hidden]
    ✔ Deployed new dataset at address 0xCb781f3106E25E2A9408C4B89C47034877223D12


Publish a dataset order
-----------------------

- Create an order template

.. code-block:: bash

    iexec order init --dataset --wallet-file developper_wallet

Edit the order part in iexec.json to describe the dataset.

===================== ==========================================================
Parameter               Meaning
===================== ==========================================================
 dataset                dataset address
 datasetprice           dataset price
 volume                 number of order created
 tag                    tag for extra computational requirement (1)
 dapprestrict:          restricted to an application defined by its address  (1)
 workerpoolrestrict     restricted to a workerpool defined by its address (1)
 requesterrestrict:     restricted to a requester defined by its address (1)
===================== ==========================================================

(1) the restriction is disabled by default with 0x0000000000000000000000000000000000000000

The volume is the total number of tasks allowed within the order created.
Once all the volume is consumed, the dataset won't be available, the dataset owner has to publish a new datasetorder:


.. code-block:: javascript

    "datasetorder": {
      "dataset": "0xCb781f3106E25E2A9408C4B89C47034877223D12",
      "datasetprice": 2,
      "volume": 1000000,
      "tag": "0x0000000000000000000000000000000000000000000000000000000000000000",
      "apprestrict": "0x0000000000000000000000000000000000000000",
      "workerpoolrestrict": "0x0000000000000000000000000000000000000000",
      "requesterrestrict": "0x0000000000000000000000000000000000000000"
    }


Sign the order

.. code-block:: bash

    iexec order sign --dataset --wallet-file data_owner_wallet
    ℹ using chain [kovan]
    ? Using wallet data_owner_wallet
    Please enter your password to unlock your wallet [hidden]
    ✔ datasetorder signed and saved in orders.json, you can share it:
    dataset:            0xCb781f3106E25E2A9408C4B89C47034877223D12
    datasetprice:       2
    volume:             1000000
    tag:                0x0000000000000000000000000000000000000000000000000000000000000000
    apprestrict:        0x0000000000000000000000000000000000000000
    workerpoolrestrict: 0x0000000000000000000000000000000000000000
    requesterrestrict:  0x0000000000000000000000000000000000000000
    salt:               0xaaae00a749e198b9f43bc89c420aaf146f3a224c8500d327c3569075eea2c2ae
    sign:               0x87f720bb9e09762257bd62561f52b22237b2982397cb8aae19e84adf8afcb4d21f758f40dcc001a5dd018aaf48ccfd59a91f3c18adcb27c414da44436bea8c931b

Publish the order

.. code:: bash

    iexec order publish --dataset --wallet-file data_owner_wallet
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


Now the dataset is available.


Input dataset encryption
------------------------

As a dataset provider, you might want to protect your dataset with encryption in order to monetize it.
Any encrypted dataset will be decrypted on worker resources with a dataset secret key retrieved from the Secret Management Service.
This dataset secret key need to be created and push by the dataset owner.
At this point, the decrypted dataset will be ready to be used by the app.

# How to encrypt a dataset

(coming soon)


Check out http://explorer.iex.ec


Go to the `Quick start`_ section to learn how to test a dapp .

.. _Quick start: /quickstart.html
