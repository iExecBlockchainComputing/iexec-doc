Secure computing
================

Overview
--------

An essential feature of the v3 of iExec is the ability to monetize all the aspects of a computation: not only the hardware resources (Workers), but also the code and the data used in the computation (owned respectively by the dApp developer and the Data Owner).

In this setting, the iExec software stack must guarantee the security of the code and data, and ensure no one can get access to them if not in the context of a blockchain-verified iExec transaction.

Given the decentralized nature of the iExec platform, where computation can be performed by any anonymous user on unknown machines,
the only way to fulfill these requirements is to take advantage of the recent developments in hardware-enforced secure execution.
Consequently iExec v3 makes use of the Intel SGX technology to create a hardware-protected Trusted Execution Environment (TEE),
where the code and data are out of reach and invisible even for a user with physical access to the execution machine.

Challenges/Requirements/Threat model
------------------------------------

Data security is an important concern and a challenging problem even in the traditional, private and centralized computing environment. In an open and decentralized computing model like the iExec platform, these challenges are compounded several times. In addition to security against a traditional attack (eg a remote attacker exploiting a software/OS vulnerability to breach into a server and leak data), iExec must solve the following technical challenges:

* The owner of the machine where the computation takes place is anonymous and unvetted. We must assume he may steal any code or data available, especially if they are valuable.
* The data must be accessible publicly: since there can’t be any synchronisation/communication between a data owner and a data user, the (encrypted) dataset must be publicly available, on a third-party server or a decentralized file system like IPFS.
* Access must be granted in an asynchronous way: once again since there is no communication between a data owner and a data recipient/user, access to the data must be granted ahead-of-time by the data owner. In particular this prevents the use of re-encryption schemes, where the data would be encrypted specifically for the recipient using his public key. On the contrary in the iExec setting the symmetric key used for the data set/ application has to be available to the vetted application (ie whose end user has paid for the data, and is running authorized code) without supervision from the data owner.

Our solution is two-fold:

* Firstly, the iExec software supports the execution of tasks inside a trusted execution environment, using Intel SGX enclave technology. The code inside the enclave is guaranteed to behave as intended. It is responsible for retrieving the data and loading the right application to process. The code and the data inside an enclave are completely out-of-reach from code outside of the enclave - even for simple read access.

                                                   .. image:: ./_images/SGX.jpg

* Second we provide an auditable (open source) `Secret Management Service <https://github.com/iExecBlockchainComputing/SMS>`_ (SMS) software, whose purpose is to securely store the keys for the different actors involved (dataset owner, computation beneficiary). This solves the problem of delegating access rights: in essence the user delegates managing access to his data to the SMS. Of course the SMS has the same security challenges as the Worker (we must ensure it runs as expected, doesn’t spill the secrets, and is not spoofed by the owner of the machine on which it runs) hence **it must also run inside an SGX enclave**. As of now there is only one SMS running on iExec's servers, but we intend to open-source the code in the near future, so that anyone will be able to run its own SMS. Data owners will then be able to choose the SMS they trust the most. The address of the chosen SMS is parametrizable in the iExec SDK.


Security guarantees
-------------------

#. Valuable code and data will be symmetrically encrypted whenever publicly available (IPFS, public repository). Decryption will only take place inside an attested SGX enclave, whose code has been authorized by the data owner.
#. Valuable code and data will be encrypted client-side, on the machine of their owner, then pushed on a public data repository. The corresponding keys will only be communicated over TLS channels established with auditable programs, running in attested enclave.
#. The Data owner may limit the use of their data to a set of applications. In this case the key will only be sent to an attested enclave whose MREnclave matches the one whitelisted by the Data Owner.
#. The Data Owner may limit the use of their data to a reduced number of end users (beneficiaries). In this case, the result of the computation will be encrypted by the Worker and only the beneficiary will be able to decrypt it.
#. At all times, keys to the data will only be accessible by:

	* The iExec SMS service (open-source iExec code, attested by Intel IAS)
	* The iExec Dapp (with an MREnclave either guaranteed by its author or by an independent auditor) white-listed by the Data provider.
	* The enclave-based Scone software component, audited by a third-party and attested by Intel IAS.


Implementation
--------------

Across the iExec software stack there are two components that are mainly concerned with the security of data: the Worker and the SMS.

The Worker is made of two parts: the In-Enclave (trusted) Worker and the untrusted, standard Worker. All data-sensitive operations (ie handling the dataset keys and decryption, and subsequent processing of the data) are handled by the In-Enclave worker. The untrusted Worker handles interaction with the blockchain and the retribution process (PoCO) but it has no access to the data.

The Secret Management Service is an independent service, hosted either by the data provider or by a third-party, that implements a keystore in a secure, remotely attested enclave. The SMS ensures the keys are accessible at all times by an authorized user, so the computation can take place without an explicit authorization from the data provider.

The Worker and the SMS both run in an SGX enclave; they are managed by an instance of the Palaemon software running in its own enclave. The Palaemont software is responsible for creating the enclave, initializing it with the iExec software and attesting it locally.

Protocol
~~~~~~~~~
Here are the steps of an end-to-end encrypted execution:

#. The Data Owner symmetrically encrypts his dataset and pushes the result on an online repository (IPFS). He then assigns a unique (Ethereum) address to this data, by creating a dedicated smart contract for it.
#. The Data Owner establishes a secure connection with the SMS of his choice. The connection is established after verifying the SMS enclave certificate that proves to the Data Owner that the other endpoint of the TLS channel is an SGX enclave running the proper SMS software.
#. The Data Owner then pushes the symmetric key and address of the encrypted data, authenticated (signed) with his Ethereum private address, to the SMS enclave (over TLS). The SMS keeps a mapping {data key ⇔ data address}.
#. When a computation is assigned to the Worker, the Worker reads the information about the task, app to run and dataset to use, on the blockchain. With the address of the dataset he is able to retrieve which SMS is holding the corresponding symmetric key.
#. The Worker then asks the Palaemon service to create an enclave, load it with the downloaded app and attest it. Palaemon delivers a TLS certificate to the enclave, authenticated with the MREnclave (ie attesting the enclave is running the intended app).
#. The Worker downloads the encrypted data and place it in a special directory reachable by the enclave Worker.
#. The enclave Worker then requests the data key to the SMS. The request is signed with the enclave private key; the request also contains the enclave certificate delivered by Palaemon (which links the public key corresponding to the enclave private key to the MREnclave of the enclave Worker). When receiving this request, the SMS retrieves the MREnclave vetted by the data owner (by reading it on the blockchain).
#. The SMS then verifies the certificate, and compare the MREnclave of the certificate with the one on the blockchain corresponding to the Dataset address. If the comparison is positive, it sends the data key (over the TLS channel)

Analysis
~~~~~~~~

Since the data is encrypted by the data owner, their confidentiality and integrity depends on two conditions:

  - **The keys don't leak during transmission to the machine where the computation occurs**
   This is ensured by symmetric encryption of the data, transmitting the keys over TLS, and by the design of the SMS, which transmits the keys only to an enclave with the right MREnclave (and thus the right application), and only for a computation whose beneficiary has been authorized by the Data Owner. By running the SMS inside an enclave and attesting it before sending it the keys, the Data Owner can make sure he is communicating with a proper SMS that will run as intended.

  - **The keys don't leak during the computation**.
   This is ensured by remotely attesting the Worker enclave and auditing the code that runs inside it, making sure the code is not malicious and won’t leak the data or its keys.

