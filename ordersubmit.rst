Task execution
==============




Using the iExec SDK CLI
-----------------------



The `iExec SDK <https://github.com/iExecBlockchainComputing/iexec-sdk/>`_ is a CLI tool that has all the commands to interact with the iExec network.

Go checkout the `iExec SDK <https://github.com/iExecBlockchainComputing/iexec-sdk/>`_ page to read its full documentation.


Using the Marketplace and the Dapp Store
----------------------------------------

Installation of your metamask wallet in your web browser is required.

Marketplace and Dapp Store allow to buy **workerpoolorders** and submit computing task in a very simple way.

| All existing dapps, published in the Dapp Store, are available in the text box for the dapp address with default arguments.
| It is also possible to fill in any existing dapp address and arguments in the dedicated textboxes.

.. image:: _images/buy_sell.png


| Visit the iExec Marketplace (https://market.iex.ec)
| Visit the Dapp store (https://dapps.iex.ec)

Using a custom smart contract ( + Callback )
--------------------------------------------

An exemple on Kovan with factorial dapp.
Prerequiste :

```
iexec --version
2.2.39
```

check you have enough ETH and RLC on Kovan to buy the workorder + dapp Price
```
iexec wallet show
```

check app factorial on Kovan
```
iexec app show 0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe
```
Clone callback-kovan branch of Poco repo
```
  git clone -b callback-kovan https://github.com/iExecBlockchainComputing/PoCo.git
  cd PoCo
  npm i
```
  * edit 'mnemonic = "12 words";' in truffle.js and paste your mnemonic admin wallet.

  It will deploy the example contract `IexecAPI <https://github.com/iExecBlockchainComputing/PoCo/blob/callback/contracts/IexecAPI.sol/>`_

  You can customize this contract to your needs (access control, store RLC on it by user, etc ...). Then deploy it with :

```
./node_modules/.bin/truffle migrate --network kovan
```
Note IexecAPI address: $IexecAPI_Adresss

A ) <b>You must sent some RLC to the IexecAPI contract</b>

```
iexec wallet sendRLC 2000 --to $IexecAPI_Adresss
```
B ) <b>You must approve the IexecAPI to send RLC to the IexecHub  :</b>

B 1) Go to https://www.myetherwallet.com/#contracts and select Network KOVAN

B 2) add the $IexecAPI_Adresss  in "Contract Address" field.

B 3) fill "ABI / JSON Interface" fieldwith contract ABI found here :https://github.com/iExecBlockchainComputing/PoCo/blob/callback-kovan/contracts/IexecAPI.abi

B 4) Click Access and select function approveIexecHub.

B 5) set amount 2000  (need to cover dappPrice + Market Order price)

B 6) unlock your admin wallet and send transaction.

check in etherscan transaction is OK and you see approve event.

`Transaction example <https://kovan.etherscan.io/tx/0x8083bb585e1414c2833d16637c96deadb0e01ec87891b69fecc8e16b26bdbf21/>`_


C ) <b>You must deposit RLC to the IexecHub through IexecAPI to be able to buy workorder (requester will be the IexecAPI contract and pay the execution): </b>

C 1) Go to https://www.myetherwallet.com/#contracts and select Network KOVAN

C 2) add the $IexecAPI_Adresss  in "Contract Address" field.

C 3) fill "ABI / JSON Interface" fieldwith contract ABI found here :https://github.com/iExecBlockchainComputing/PoCo/blob/callback-kovan/contracts/IexecAPI.abi

C 4) Click Access and select function depositRLCOnIexecHub.

C 5) set amount 2000

C 6) unlock your admin wallet and send transaction.


check in etherscan transaction is OK and you see Deposit event.

`Transaction example <https://kovan.etherscan.io/tx/0x378ad8c8da3c4463ad9decca4a4974dd6eeba53cea444a155db2d0578bdfeb91/>`_

D )<b> You can now buyForWorkOrder on the IexecAPI contract :</b>

D 1) Go to https://www.myetherwallet.com/#contracts and select Network KOVAN

D 2) add the $IexecAPI_Adresss  in "Contract Address" field.

D 3) fill "ABI / JSON Interface" fieldwith contract ABI found here :https://github.com/iExecBlockchainComputing/PoCo/blob/callback-kovan/contracts/IexecAPI.abi

D 4) Click Access and select function buyForWorkOrder.

D 5) set params as follow :
 _marketorderIdx : set one found in the marketplace

_workerpool : set the workerpool address of the _marketorderIdx selected

_app : 0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe

_dataset : 0x0000000000000000000000000000000000000000

_params :{"cmdline": "10"}

_callback : the $IexecAPI_Adresss

_beneficiary : your wallet or the wallet that is allowed to download the result.

D 6) set gas estimate to 972397

D 7) unlock your admin wallet and send transaction.

A buyForWorkOrder transaction successful example :

`Transaction example <https://kovan.etherscan.io/tx/0xb465f9980848f030526035812181263f332fdefe9577aa3e1a7fdda08c2330f9/>`_

Watch the workorder (found woid in the transaction previous Log) :

change 0xe16ada2d83021632cd78a2fbf7620ce485064365 with your woid found.

```
iexec work show 0xe16ada2d83021632cd78a2fbf7620ce485064365 --watch
```

note : You must see :

 * m_requester  : is your smart contract IexecAPI address. it has pay the execution.
 * m_callback   : is your smart contract IexecAPI address. it will receive the callback.
 * m_beneficiary : is your or the wallet that will be able to download the result.

Then, wait for workorder m_statusName is COMPLETED.
Check that the callback has been done on your contract.
Successful workOrderCallback tx factorial 10 example :

`Transaction example <https://kovan.etherscan.io/tx/0x562094cf17e83d4c8e8f6d0a05e8a742f88270d37c77e977e6d75160deb6c72c#eventlog/>`_

And Beneficiary can also download the result too :

```
MBPdefrancois2:call fbranci$ iexec work show   0xe16ada2d83021632cd78a2fbf7620ce485064365 --download
ℹ using chain [kovan]
✔ work 0xe16ada2d83021632cd78a2fbf7620ce485064365 status is COMPLETED, details:
m_workerpool:          0x82190e18f7ce7cb9d39128707f58d19c649cf9c2
m_params:              {"cmdline": "10"}
m_requester:           0xf1b2550e4ea1c4ffae1dfb790948c895614e4457
m_app:                 0x2f185a1e5ced207d64d9c94e39c0f060c38fc2fe
m_dataset:             0x0000000000000000000000000000000000000000
m_emitcost:            1
m_uri:                 xw://api-bench-pool.iex.ec/d17d7bc7-ce85-4cfd-aeea-40ace83e9f89
m_stdout:
  """
    3628800

  """
m_resultCallbackProof: 0xe5cb7d00b38206b597110444d4da0600448c754511a43c341a92dab2a99cc061
m_iexecHubAddress:     0x12b92a17b1ca4bb10b861386446b8b2716e58c9b
m_callback:            0xf1b2550e4ea1c4ffae1dfb790948c895614e4457
m_status:              4
m_marketorderIdx:      1437
m_stderr:
m_beneficiary:         0x486a5986f795d323555c0321d655f1eb78d68381
m_statusName:          COMPLETED

✔ downloaded work result to file /Users/fbranci/iexecdev/call/0xe16ada2d83021632cd78a2fbf7620ce485064365.text
MBPdefrancois2:call fbranci$ cat 0xe16ada2d83021632cd78a2fbf7620ce485064365.text
3628800
```
