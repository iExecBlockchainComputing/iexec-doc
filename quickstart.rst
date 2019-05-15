Quick start
===========


iExec, being blockchain-based, allows you to manage your computing transactions in a secure and decentralized environment.
In order to transact between multiple parties within iExec, an Ethereum wallet is required.
First, you will need to create your Ethereum wallet and credit it with Ethereum tokens (ETH), before running the application.
While iExec is in its development phase, we allow transactions on Ethereum’s Kovan Network. Sometimes referred to as test network, it uses Kovan ETH tokens that hold no real value and are used solely for testing purposes.


SDK Installation
----------------

Prerequisites:
 - npm
 - zip


.. code-block:: bash

    sudo npm -g -i install iexec


- Now you need to create configuration files

.. code-block:: bash

    iexec init

    ℹ Here is your main config "iexec.json":
    description: My iExec ressource description, must be at least 150 chars long in order to pass the validation checks. Describe your application, dataset or workerpool to your users
    license:     MIT
    author:      ?
    social:
      website: ?
      github:  ?
    logo:        logo.png

    ℹ Here is your chain config "chain.json":
    default: kovan
    chains:
      dev:
        host: http://localhost:8545
        sms:  http://localhost:5000
        id:   17
        hub:  0x60E25C038D70A15364DAc11A042DB1dD7A2cccBC
      ropsten:
        host: https://ropsten.infura.io/v3/f3e0664e01504f5ab2b4360853ce0dc7
        id:   3
      rinkeby:
        host: https://rinkeby.infura.io/v3/f3e0664e01504f5ab2b4360853ce0dc7
        id:   4
      kovan:
        host: https://kovan.infura.io/v3/f3e0664e01504f5ab2b4360853ce0dc7
        id:   42
      mainnet:
        host: https://mainnet.infura.io/v3/f3e0664e01504f5ab2b4360853ce0dc7
        id:   1

    ℹ Creating your wallet file
    ? Please choose a password for wallet encryption [input is hidden]
    ? Please confirm your password [hidden]
    ℹ Your wallet address is 0x10C3580A88E3B7400e4B8D0F046CD66D21482C24 wallet file saved in "/root/.ethereum/keystore/UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24" you must backup this file safely :

    address: 10c3580a88e3b7400e4b8d0f046cd66d21482c24
    id:      2ae0905e-e102-452f-927b-349eafc9cd87
    version: 3
    Crypto:
      cipher:       aes-128-ctr
      cipherparams:
        iv: 5117269c592c935eeb7367507732ef56
      ciphertext:   08d2b7f8790f167ed9e4179dc3b03952a95fa196d3e5a6ea79274063ad14b817
      kdf:          scrypt
      kdfparams:
        salt:  4d9a74104f0822927ae81e41702a1d56a6d770c57b11aadcec3bdd9beb972eeb
        n:     131072
        dklen: 32
        p:     1
        r:     8
      mac:          96eaa6a86d0ab1a42fb3578f0e98f59be6861294c971c1e481e4bbded85adf67

    ⚠ You must backup your wallet file in a safe place!
    ✔ iExec project is ready




The SDK stores the encrypted wallet in '~/ethereum/keystore' directory.

.. NOTE::
    - You can manage multiple wallets with  **--wallet-file** and **--wallet-address** options. Example : iexec wallet show --wallet-file user_wallet

    - You can import an existing wallet with "**iexec wallet import**" command

Check your wallet balance with

.. code-block:: bash

    iexec wallet show
    ? Using wallet UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24
    Please enter your password to unlock your wallet [hidden]
    ℹ Wallet file:
    publicKey: 0x785f93e1bf8f9657d6cd63f2bd6c8189ece95ceec4772afdbbb81da995ba9a963fb4ebe0633c8925fe99d351580ccbeb2d3d74bfdef6c4072387a4c3729f7dd9
    address:   0x10C3580A88E3B7400e4B8D0F046CD66D21482C24

    ℹ using chain [kovan]
    ✔ Wallet kovan balances [42]:
    ETH:  0
    nRLC: 0


- Get ETH:

 a small amount of ether cryptocurrency (ETH) is necessary to interact with the ethereum blockchain.

.. code-block:: bash

    iexec wallet getETH


.. NOTE::
    | **Alternative method**
    |
    | Post your your public ethereum address in https://gitter.im/kovan-testnet/faucet
    | You will receive a small amount of ETH in few minutes.


- Get RLC:

 RLC can be used to reward the computing resources.

.. code-block:: bash

    iexec wallet getRLC
    ℹ using chain [kovan]
    ? Using wallet UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24
    Please enter your password to unlock your wallet [hidden]
    ✔ Faucets responses:
    - faucet.iex.ec :
    ok:      true
    message: Successfully requested 200 nRLC. Your position in the waiting list is 1/10


Check the wallet is now charged with ETH and RLC

.. code-block:: bash

    iexec wallet show
    ? Using wallet UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24
    Please enter your password to unlock your wallet [hidden]

    ℹ Wallet file:
    publicKey: 0x785f93e1bf8f9657d6cd63f2bd6c8189ece95ceec4772afdbbb81da995ba9a963fb4ebe0633c8925fe99d351580ccbeb2d3d74bfdef6c4072387a4c3729f7dd9
    address:   0x10C3580A88E3B7400e4B8D0F046CD66D21482C24

    ℹ using chain [kovan]
    ✔ Wallet kovan balances [42]:
    ETH:  0.1
    nRLC: 200


.. include:: paypertask.rst


Task execution
--------------


Let's execute the vanitygen application, a command-line vanity ethereum address generator starting with a given pattern.
The address can be found in https://explorer.iex.ec/kovan application deployed tab.


- Deposit RLC on your account

Now you have to charge your account.
Your wallet, for security purpose, is never directly connected to the market,
you have to charge your account to allow deal's smart contract to lock fund engaged to pay the task.


.. code-block:: bash

    iexec account deposit 40
    ℹ using chain [kovan]
    ? Using wallet UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24
    Please enter your password to unlock your wallet [hidden]
    ✔ deposited 40 nRLC to your iExec account


- We need to find a existing order for the application, we will use the orderhash for the task submission. The application is registered on the blockchain, and app order has to be filled to run the task.

.. code-block:: bash

     iexec orderbook app 0x4C042521b6DaD59558b0B5bE939881e98054bD34
    ℹ using chain [kovan]
    ✔ Orderbook
    app orders details:
    -
      orderHash:            0xe48ac12eefdfab06486e80ab2dfdd2caef6c293de80262b8a5845d653785a396
      app:                  0x4C042521b6DaD59558b0B5bE939881e98054bD34
      price:                0
      remaining:            999996
      publicationTimestamp: 2019-04-24T09:24:19.037Z

Make sure apporder still remains for the application. see **remaining** field.

Copy the orderHash, it will be used later.


- Find a **workerpoolorder** in the orderbook

The workerpoolorder corresponds to the resources you can use to run your task.
Many orders are availables, selection can be based regarding your preference: price or worker pool location.


.. code-block:: bash

    iexec orderbook workerpool --category 4
    ℹ using chain [kovan]
    ✔ Orderbook
    workerpool orders details:
    -
      orderHash:            0xafe84857ae390005f3d5566a4fd6c0b336ca4dd00963e8ce67dd01fc8c4089d6
      workerpool:           0x7Cb70eB2e99Efd5Ce7ca2d6899cE81D1ed46E770
      category:             4
      trust:                1
      price:                35
      remaining:            1
      publicationTimestamp: 2019-04-24T11:57:50.708Z
    -
      orderHash:            0x615394b1c3c6b845416ea00fa799bf7d6b42157db23a87ab1c4b52bf6b91cbf0
      workerpool:           0x7Cb70eB2e99Efd5Ce7ca2d6899cE81D1ed46E770
      category:             4
      trust:                1
      price:                36
      remaining:            1
      publicationTimestamp: 2019-04-24T12:00:06.977Z
    -
      orderHash:            0xf0fd940264500d16260ef42f33a914232c1362f8dee54c2f8fb87926d57655ab
      workerpool:           0x7Cb70eB2e99Efd5Ce7ca2d6899cE81D1ed46E770
      category:             4
      trust:                1
      price:                40
      remaining:            1
      publicationTimestamp: 2019-04-24T12:01:16.696Z
    -
      orderHash:            0xb8445c69231330642b0bf3e160257c868f935e90b38ded743194c02a21ebc038
      workerpool:           0x7Cb70eB2e99Efd5Ce7ca2d6899cE81D1ed46E770
      category:             4
      trust:                1
      price:                50
      remaining:            1
      publicationTimestamp: 2019-04-24T13:12:22.171Z

Select your order and copy the related orderHash

- Create a task template

.. code-block:: bash

   iexec order init --request
    ℹ using chain [kovan]
    ✔ Saved default requestorder in "iexec.json", you can edit it:
    app:                0x0000000000000000000000000000000000000000
    appmaxprice:        0
    dataset:            0x0000000000000000000000000000000000000000
    datasetmaxprice:    0
    workerpool:         0x0000000000000000000000000000000000000000
    workerpoolmaxprice: 0
    volume:             1
    category:           0
    trust:              0
    tag:                0x0000000000000000000000000000000000000000000000000000000000000000
    beneficiary:        0x10C3580A88E3B7400e4B8D0F046CD66D21482C24
    callback:           0x0000000000000000000000000000000000000000
    params:             { "0": "--help" }
    requester:          0x10C3580A88E3B7400e4B8D0F046CD66D21482C24


Edit the task description in the iexec.json file.

Fill in the app address, the category, a maximum price for the workerpool order and task's parameter.

Limitation price exists also for dataset and app but the current example use free of charge application and no dataset.

.. code-block:: bash

    ...
    "requestorder": {
      "app": "0x4C042521b6DaD59558b0B5bE939881e98054bD34",
      "appmaxprice": 0,
      "dataset": "0x0000000000000000000000000000000000000000",
      "datasetmaxprice": 0,
      "workerpool": "0x0000000000000000000000000000000000000000",
      "workerpoolmaxprice": 100,
      "volume": 1,
      "category": 0,
      "trust": 0,
      "tag": "0x0000000000000000000000000000000000000000000000000000000000000000",
      "beneficiary": "0x10C3580A88E3B7400e4B8D0F046CD66D21482C24",
      "callback": "0x0000000000000000000000000000000000000000",
      "params": "{ \"0\": \"1iEx\" }",
      "requester": "0x10C3580A88E3B7400e4B8D0F046CD66D21482C24"
    }


- Create the request order

.. code-block:: bash

    iexec order sign --request
    ℹ using chain [kovan]
    ? Using wallet UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24
    Please enter your password to unlock your wallet [hidden]
    ✔ requestorder signed and saved in orders.json, you can share it:
    app:                0x4C042521b6DaD59558b0B5bE939881e98054bD34
    appmaxprice:        0
    dataset:            0x0000000000000000000000000000000000000000
    datasetmaxprice:    0
    workerpool:         0x0000000000000000000000000000000000000000
    workerpoolmaxprice: 0
    volume:             1
    category:           0
    trust:              0
    tag:                0x0000000000000000000000000000000000000000000000000000000000000000
    beneficiary:        0x10C3580A88E3B7400e4B8D0F046CD66D21482C24
    callback:           0x0000000000000000000000000000000000000000
    params:             { "0": "1iEx" }
    requester:          0x10C3580A88E3B7400e4B8D0F046CD66D21482C24
    salt:               0x7dfe93a03528fad9707cea738da11ae36d64ee3244b85bf00f3fd7d21899ba38
    sign:               0x83b6d9cd4e1e6e7279c599852e26b821d0c8b0210f595dc14e699c432bceab6d719645b1ead98e851585187a7b6e234514008e3271b049261a61b7f71239ad081b

Now the request order is created and it can be submitted

.. code-block:: bash

    iexec order fill --workerpool 0xf0fd940264500d16260ef42f33a914232c1362f8dee54c2f8fb87926d57655ab --app 0xe48ac12eefdfab06486e80ab2dfdd2caef6c293de80262b8a5845d653785a396
    ℹ using chain [kovan]
    ℹ Fetching apporder 0xe48ac12eefdfab06486e80ab2dfdd2caef6c293de80262b8a5845d653785a396 from iexec marketplace
    ℹ Fetching workerpoolorder 0xf0fd940264500d16260ef42f33a914232c1362f8dee54c2f8fb87926d57655ab from iexec marketplace
    ? Using wallet UTC--2019-04-24T10-11-45.987000000Z--10C3580A88E3B7400e4B8D0F046CD66D21482C24
    Please enter your password to unlock your wallet [hidden]
    ✔ 1 task successfully purchased with dealid 0xccb19c6792b6e9da0ea9d5af46a221e6836da93cce57cdfd8014c900b695b691



| The command returns the dealid address.
| Let's retrieve the task id

.. code-block:: bash

    iexec deal show 0xccb19c6792b6e9da0ea9d5af46a221e6836da93cce57cdfd8014c900b695b691 --tasks 0
    ℹ using chain [kovan]
    ✔ Deal 0xccb19c6792b6e9da0ea9d5af46a221e6836da93cce57cdfd8014c900b695b691 details:
    app:
      pointer: 0x4C042521b6DaD59558b0B5bE939881e98054bD34
      owner:   0xA1162f07afC3e45Ae89D2252706eB355F6349641
      price:   0
    dataset:
      pointer: 0x0000000000000000000000000000000000000000
      owner:   0x0000000000000000000000000000000000000000
      price:   0
    workerpool:
      pointer: 0x7Cb70eB2e99Efd5Ce7ca2d6899cE81D1ed46E770
      owner:   0xE9E34b370B67F1285C1DCD9a631c11A20fE65Ef3
      price:   40
    trust:                1
    category:             4
    tag:                  0x0000000000000000000000000000000000000000000000000000000000000000
    requester:            0x10C3580A88E3B7400e4B8D0F046CD66D21482C24
    beneficiary:          0x10C3580A88E3B7400e4B8D0F046CD66D21482C24
    callback:             0x0000000000000000000000000000000000000000
    params:               { "0": "1iEx" }
    startTime:            1556115748
    botFirst:             0
    botSize:              1
    workerStake:          12
    schedulerRewardRatio: 1

    Tasks:
    -
      idx:    0
      taskid: 0x0e617507e446658383918f2dbe14f2e65e161ac47b131eab46ecba50dfe26ae5


- Monitor your work

.. code-block:: bash

     iexec task show 0x8374e2d96305a4a9b3f84e531b67e350f008b31d
    ℹ using chain [kovan]
    ✔ work 0x8374e2d96305a4a9b3f84e531b67e350f008b31d status is ACTIVE, details:
    m_workerpool:          0x0061b8b1191394fa710def946368675b79db062b
    m_params:              {"cmdline":"7"}
    m_requester:           0x2e1d3f65d6d09f8aa7661e3e810d6a77a4da3869
    m_app:                 0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe
    m_dataset:             0x0000000000000000000000000000000000000000
    m_emitcost:            1
    m_uri:
    m_stdout:
    m_resultCallbackProof: 0x0000000000000000000000000000000000000000000000000000000000000000
    m_iexecHubAddress:     0x12b92a17b1ca4bb10b861386446b8b2716e58c9b
    m_callback:            0x0000000000000000000000000000000000000000
    m_status:              1
    m_marketorderIdx:      1884
    m_stderr:
    m_beneficiary:         0x0000000000000000000000000000000000000000
    m_statusName:          ACTIVE

    ℹ if work is not "COMPLETED" after Thu Jan 03 2019 03:05:36 GMT+0100 (CET) you can claim the work to get a full refund using "iexec work claim"

until it is completed

.. code-block:: bash

    .....
    m_statusName:          COMPLETED

- Download the result and check your result

.. code-block:: bash

    iexec task show  0xafaa1b9c1ae6b46a0d462d288d0147f84bd011a07ddd93908d37d32256804216 --watch --download
    ℹ using chain [kovan]
    ? Using wallet UTC--2019-04-25T09-56-05.957000000Z--849dC529c01ddf5FFdD52F411dBD38259D83Aeec
    Please enter your password to unlock your wallet [hidden]
    ℹ Task status COMPLETED
    ✔ Task 0xafaa1b9c1ae6b46a0d462d288d0147f84bd011a07ddd93908d37d32256804216 details:
    status:               3
    dealid:               0xb610ed792fa222c72e6e7482b20da3246102bb89c22f2b7f0e2dce251b91616d
    idx:                  0
    timeref:              36000
    contributionDeadline: 1556446320
    revealDeadline:       1556266372
    finalDeadline:        1556554320
    consensusValue:       0xc7f1cea6a10b0fe00966cbc01b7d5d3c728563c01df9e69d5c2e80794043b0c9
    revealCounter:        1
    winnerCounter:        1
    contributors:
    resultDigest:         0x72f4acbfdc93499509441507db54efc04d6744a924910549b7e96a207daa4b0c
    results:              https://jupiter-pool.iex.ec:443/results/0xafaa1b9c1ae6b46a0d462d288d0147f84bd011a07ddd93908d37d32256804216
    statusName:           COMPLETED

    ℹ downloaded task result to file /0xafaa1b9c1ae6b46a0d462d288d0147f84bd011a07ddd93908d37d32256804216.zip

.. code-block:: bash

     unzip /0xafaa1b9c1ae6b46a0d462d288d0147f84bd011a07ddd93908d37d32256804216.zip
     Archive:  /0xafaa1b9c1ae6b46a0d462d288d0147f84bd011a07ddd93908d37d32256804216.zip
     inflating: stdout.txt


.. code-block:: bash

    cat stdout


**Result encryption**

You can choose to get an encrypted result depending on the privacy of your task result.

* If the beneficiary of the task is set, the result will be pushed to the iExec Result Repository:
  - If the key of the beneficiary has been pushed to the Secret Management Service, the result will be encrypted before being pushed to the Result Repository. The beneficiary of the task will be the only one able to access and decrypt the result.
  - If the key of the beneficiary is missing in the Secret Management Service, the result will be pushed to the Result Repository without encryption. The beneficiary of the task will be the only one able to access the result.
* If the beneficiary of the task is unset, the result will be pushed to IPFS without encryption.


# How run a task with an encrypted result

1. Generate your beneficiary keys

.. code:: bash

    iexec tee generate-beneficiary-keys

2. Push your keys to the SMS

Please check your 'chain.json' file contains an entry '"sms": "https://kovan-pool.iex.ec:443"'

.. code:: bash

    iexec tee push-secret

3. Buy computation with your beneficiary address

market.iex.ec: Advanced parameters > Beneficiary > Private
SDK: iexec.json > requestorder > beneficiary > 0xyourAddress

4. Download the result and decrypt it

.. code:: bash

    iexec task show <0xtask> --download
    iexec tee decrypt-results <encryptedResultsPath>


.. include:: contactus.rst
