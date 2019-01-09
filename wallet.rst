Wallet Management
=================

| The cryptocurrency wallet is a secure software program to interact with blockchains.
| The wallet enables user to send and receive digital currency and monitor their balance and conduct other operations as smart contract.

| iExec service runs on ethereum blockchain, the deal between agents (data owner, resources provider, requestor, developer) are made using RLC token.

| This transaction execution (smart contract) takes some amount of gas in ETH,
 this gas is used to calculate the amount of fees that need to be paid to the network in order to execute an operation.


Create your Wallet
------------------

If you haven’t already, you will need to create an Ethereum wallet:

* Go to MyEtherWallet https://myetherwallet.com
* Create a wallet following MEW’s instructions

.. WARNING::
    Remember your password. You will need it later.
    Back up your wallet properly.

.. figure:: _images/mew_createwallet.png

* Download your Keystore file, **keep it safe and secret**.

* To Import the wallet using the Keystore file, you will need to open the downloaded wallet file with any text editor and copy its content.

Useful links:
  How to get an Ethereum wallet ? https://kb.myetherwallet.com/offline/running-myetherwallet-locally.html


Credit your wallet with ETH and RLC
-----------------------------------

| Your wallet must be credited with both ether (ETH) and RLC.
| Ether is used to process the smart contracts.

Useful links:
 - How to get some test ETH (Kovan) on your Ethereum wallet ? https://gitter.im/kovan-testnet/faucet
 - How to get some test RLC (Kovan) on your Ethereum wallet ? https://faucet.iex.ec/kovan


Login with Metamask
-------------------

| Once you’ve unlocked your Metamask, you can login on the iExec marketplace https://market.iex.ec with embedded wallet management.
| In the **Account**, both Balance and Allowance are displayed, and values are expressed in nRLC (Nano RLC).
| When a computing deal is closed, a dedicated smart contract is created,
 an allowance step is mandatory to give authority to this new smart contract proceed to the payment when the computing task is successfully ended.

Set up the SDK with your own wallet
-----------------------------------

Edit the wallet.json file with your own information

Go checkout the `iExec SDK <https://github.com/iExecBlockchainComputing/iexec-sdk/>`_ page to get wallet management option.


Stacking and incentives
-----------------------

.. warning:: Introduce the concepts

+---------------------+----------------+------------------+------------------+
|    **Role**         | **RLC needed** | **ETH needed**   |    **Stacking**  +
+---------------------+----------------+------------------+------------------+
| requestor           |   xxx          |    xxx           |    xxx           |
+---------------------+----------------+------------------+------------------+
| dapp provider       |   xxx          |    xxx           |    xxx           |
+---------------------+----------------+------------------+------------------+
| resources provider  |   xxx          |    xxx           |    xxx           |
+---------------------+----------------+------------------+------------------+





