Getting started
===============

| iEXec, as a blockchain based solution, allows you to manage your computing transactions in a secure and decentralized environment.
| This transactions between many parties require an ethereum wallet to interact.
| You will first create and credit your ethereum wallet and then run an application.
| The test occurs on kovan network, one of the testing networks of ethereum, with no real value.


SDK Installation
----------------

.. code-block:: bash

    sudo npm -g -i iexec


- Create all the configuration files **in the current directory**,

.. code-block:: bash

    iexec init

    ✔ wallet saved in "wallet.json":

    privateKey: 0xc0be66b209f411c9b8a2eb0abc3dd90414f8349d23d01a06e56b05ed83564706
    publicKey:  0xcef04083fc9f537ee19b17fd1c4431d55172e7ba0e153802a641ec0f268a5d1466599b5a90cd0dd2a34fe34359f4ad044d8e0256b306c3e4f011ec26e32d1bbe
    address:    0x403e633A31b01aFd2BeAadb19764f0eBa7F3c059

The SDK creates for you a wallet and store all your wallet information in wallet.json file.

.. NOTE::
    Encrypted wallet option is available,
    check `iExec SDK documentation <https://github.com/iExecBlockchainComputing/iexec-sdk/>`_ for more details


Check your wallet balance with

.. code-block:: bash

    iexec wallet show
    ℹ using chain [kovan]
    ℹ Wallet file:
    privateKey: 0x718d1aabe144f93edf8c9645c601ebccd02a083d36868a085d6bcab84bcd965f
    publicKey:  0xa7bbb3eb94abbc6a9050d4d11fbc0b1cff0a03ee84a3f1929a0d232506daf32c58a5a477c786e01e1a9476cc50b819a134b9dbe2c61eb36700ddba938e8dbbe2
    address:    0x6818ea75a8560e1823fe6db4aca931d492432c5f

    ✔ Wallet kovan balances [42]:
    ETH:  0
    nRLC: 0


- Get ETH: a small amount of ether cryptocurrency (ETH) is necessary to interact with the ethereum blockchain.

.. code-block:: bash

    iexec wallet getETH


.. NOTE::
    | **Alternative method**
    |
    | Post your your public ethereum address in https://gitter.im/kovan-testnet/faucet
    | You will receive a small amount of ETH in few minutes.


- Get RLC: RLC can be used to reward the computing resources.

.. code-block:: bash

    iexec wallet getRLC
    ℹ using chain [kovan]
    ✔ Faucets responses:
    - faucet.iex.ec :
    {
        "ok": true,
        "message": "Successfully requested 200 nRLC. Your position in the waiting list is 1/10"
    }


Task execution
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


To validate the installation, let's launch the factorial application. Given N, it computes N!

- Edit the task description in the iexec.json file.

The factorial app is defined by its ethereum address.
We also set up the parameter of the application N=7

.. code-block:: bash

    {
      "order": {
        "buy": {
          "app": "0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe",
          "dataset": "0x0000000000000000000000000000000000000000",
          "params": {
            "cmdline": "7"
          }
        }
      }
    }

- Find a **workerpoolorder** in the orderbook

The workerpoolorder corresponds to the resources you can buy to run your task.

.. code-block:: bash

     iexec orderbook show
    ℹ using chain [kovan]
    ✔ orderbook details:
    -
      id:        1884
      price:     12315
      pool:      0x0061B8b1191394FA710Def946368675B79DB062b
      category:  5
      timestamp: 2018-12-21T12:01:24.000Z
    -
      id:        1880
      price:     13994
      pool:      0x49327538C2f418743E70Ca3495888a62B587A641
      category:  5
      timestamp: 2018-12-20T15:53:36.000Z
    -
      id:        1859
      price:     14882
      pool:      0x49327538C2f418743E70Ca3495888a62B587A641
      category:  5
      timestamp: 2018-12-20T15:01:24.000Z
    -
      id:        1898
      price:     15978
      pool:      0x49327538C2f418743E70Ca3495888a62B587A641
      category:  5
      timestamp: 2018-12-30T19:29:36.000Z
    -
      id:        1883
      price:     17708
      pool:      0x49327538C2f418743E70Ca3495888a62B587A641
      category:  5
      timestamp: 2018-12-21T11:58:44.000Z

    ℹ trade in the browser at https://market.iex.ec


- Deposit RLC on your account

Now you have to charge your account


.. code-block:: bash

    iexec account deposit 15000
    ℹ using chain [kovan]
    ✔ deposited 15000 nRLC to your iExec account


- Fill the order

.. code-block:: bash

    iexec order fill 1884
    ℹ using chain [kovan]
    ℹ app price: 1 nRLC for app 0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe
    ℹ workerpool price: 12315 nRLC for workerpool 0x0061b8b1191394fa710def946368675b79db062b
    ℹ work parameters:
    cmdline: 7

    ? Do you want to spend 12316 nRLC to fill order with ID 1884 and submit your work Yes
    ✔ Filled order with ID 1884
    ✔ New work at 0x8374e2d96305a4a9b3f84e531b67e350f008b31d submitted to workerpool 0x0061b8b1191394fa710def946368675b79db062b

The command returns the ethereum work address.


- Monitor your work

.. code-block:: bash

     iexec work show 0x8374e2d96305a4a9b3f84e531b67e350f008b31d
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

- Download the result and check the result of 7! (factorial(7))

.. code-block:: bash

     iexec work show 0x8374e2d96305a4a9b3f84e531b67e350f008b31d --download
    ℹ using chain [kovan]
    ✔ work 0x8374e2d96305a4a9b3f84e531b67e350f008b31d status is COMPLETED, details:
    m_workerpool:          0x0061b8b1191394fa710def946368675b79db062b
    m_params:              {"cmdline":"7"}
    m_requester:           0x2e1d3f65d6d09f8aa7661e3e810d6a77a4da3869
    m_app:                 0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe
    m_dataset:             0x0000000000000000000000000000000000000000
    m_emitcost:            1
    m_uri:                 xw://api-ibm-pool.iex.ec/1faad140-f38b-4bc0-b66c-dda8fefec4f6
    m_stdout:
    m_resultCallbackProof: 0x67bfbf015f2d7726eb9e636060cbaaaacf2ac45479293410f4fb22586bcdbb0e
    m_iexecHubAddress:     0x12b92a17b1ca4bb10b861386446b8b2716e58c9b
    m_callback:            0x0000000000000000000000000000000000000000
    m_status:              4
    m_marketorderIdx:      1884
    m_stderr:
    m_beneficiary:         0x0000000000000000000000000000000000000000
    m_statusName:          COMPLETED

    ✔ downloaded work result to file /tmp/0x8374e2d96305a4a9b3f84e531b67e350f008b31d.text

.. code-block:: bash

     cat /tmp/0x8374e2d96305a4a9b3f84e531b67e350f008b31d.text
    5040

Installation is complete.


For technical support, contact us:
  - mail support@iex.ec
  - slack iexec-team-private.slack.com
