Technical Reports
-----------------

- Trust management in the Proof of Contribution protocol in V3    `PDF <https://github.com/iExecBlockchainComputing/iexec-doc/raw/master/techreport/iExec_PoCo_and_trustmanagement_v1.pdf>`_
- Nominal workflow.                                          `PNG <https://github.com/iExecBlockchainComputing/iexec-doc/raw/master/techreport/nominalworkflow-ODB.png>`_

Releases
--------
This section is for the different releases made. The latest stable version is 2.3.

Version 2.3 "Gold Lancer” (金槍手)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The iExec v2.3 labelled “Gold Lancer” (金槍手) sees the addition of an SDK that provides full end-to-end privacy-preserving computing, guarantees execution integrity, and provides an on-chain enclave execution attestation. It consists of the following releases of the different modules:

========  =======  ========================================================================
Module    Version  Link
========  =======  ========================================================================
XtremWeb  14.0.0   `<https://github.com/iExecBlockchainComputing/xtremweb-hep/releases/tag/14.0.0>`_
SDK       2.2.39   `<https://github.com/iExecBlockchainComputing/iexec-sdk/releases/tag/v2.2.39>`_
PoCo      1.0.14   `<https://github.com/iExecBlockchainComputing/PoCo/>`_
========  =======  ========================================================================


Glossary
--------

.. glossary::


Ethereum:
    Ethereum is an open-source, public, blockchain-based distributed computing platform and operating system featuring smart contract functionality.

Smart Contract:
    Smart contracts help you exchange anything of value in a transparent, conflict-free way without needing the services of a middleman or third party. It is a computer protocol intended to digitally facilitate the negotiation of a contract. Smart contracts use blockchain to automate consensus over transactions. The transactions within these contracts are fully publicly auditable, traceable and irreversible.

Dapp:
    Dapp is an abbreviated form for decentralized application. For a dapp, the backend code is based on a decentralized peer-to-peer network. For example, dapps can be based on Ethereum smart contracts. Contrast this a with a normal app, where the backend code runs on centralized servers.
    iExec Dapps are powered by the decentralized cloud infrastructure with machines distributed over the Ethereum network. Rather than relying on computing power from a single centralized data center.

Dataset:
    A dataset is a collection of related sets of information that is composed of separate elements, such as numbers, semantic-data or variables, that can be manipulated by a computer for practical application. For example, iExec data to be used within the medial industry can be use healthcare professionals, care providers, insurers, and government agencies.

Task:
    A task within iExec is an instance where computing power is required.

RLC:
    RLC tokens will be used to access the resources provided through iExec. It is the unique method of payment between application providers, server providers and data providers.

Requester:
    An individual or enterprise requesting the use of cloud resources through iExec.

Worker:
    They are individuals or companies who own computing resources and are willing to make them available for the computation of tasks against payments in RLC.

Worker Pool:
    Worker pools organize the contributions of Workers.  A worker pool is a group of machines, often with similar characteristics, that is led by a Pool Manager.

Pool Manager:
    A Pool Manager, who organizes the work distribution and the order publication of a Worker Pool.
    While not doing the actual computation, they receive a fee for running a scheduler machine and managing the infrastructure.  They are in competition with other pools to attract workers. They aim to achieve efficient management which guarantees the income of workers.

Proof-of-Contribution (PoCo):
    PoCo is the is the protocol used by iExec for consensus over off-chain computation. When a worker executed a computation, the result is then brought back ‘on-chain’ where PoCo carries out a blockchain-level consensus over the result to prove it has not been falsified. As well as providing trust on the platform, PoCo also orchestrates the different contributions to the iExec platform so that payment is always fair.

Staking (of RLC):
    Staking is a mechanism in PoCo that involves a certain amount of Workers’ RLC being ‘locked-up’ during the execution of a task. To prevent malicious workers, the locked RLC is staked as a security deposit.  Workers who computed a false result will lose their stake.

Intel SGX:
    Intel SGX is used by iExec to secure computations and data running on untrusted machines over the distributed and decentralized network.


Links
-----
- The iExec Dapp Store: https://dapps.iex.ec
- The iExec Marketplace: https://market.iex.ec
- The iExec Explorer: https://explorer.iex.ec
- The iExec Pools registry: https://pools.iex.ec
- The RLC faucet: https://faucet.iex.ec
- iExec main documentation: https://docs.iex.ec
- https://github.com/iExecBlockchainComputing : source code, SDK, dapps registry to apply for Dapp Store listing.


Contact us
----------

- Slack: https://slack.iex.ec
- Gitter : https://gitter.im/iExecBlockchainComputing/Lobby
- Email : support@iex.ec

