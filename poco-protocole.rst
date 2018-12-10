Protocole
=========

Objectives
----------

PoCo stands for proof of contribution. It's protocol designed to provide a trusted computing environment on top of untrusted resources. In addition to providing trust PoCo also orchestrates the different contributions to the iExec platform so that payment is always fare.

PoCo is modular. It comes with features that can be used depending on the context:

- **Result certification**

  The PoCo provides a layer of that relies on replication to achieve result certification. This is a purely software solution that enforces a confidence level on the result. This confidence level can be customized by the requester. (TODO: link to the relevant section).

  This layer also support the onchain certification of TEE (such as Intel SGX)

- **Secure payment**

  User funds are locked to ensure workers are payed. Workers have to achieve consensus to get the user's funds otherwize user is reimbursed. Worker and scheduler have to stake to participate. Bad behaviour from a actor result in a loss of stake.

  This is essential the public blockchain but all values can be set to 0 for private blockchain solutions.

- **Permissioning**

  For an execution to happen, a deal must be signed between the different parties. A permission mechanism can be used to control acces to applications, datasets and workerpools.

  This can be used by private blockchains that do not use the secure payment layer. It can also be used in the context of the public blockchain to increase the security (dataset resctricted to a specific application for example).


Overview
--------

The PoCo describes the succession of contributions that are requires to achieve consensus on a given result. The logic is details in two blog articles:

- `PoCo series #1: Initial PoCo description <https://medium.com/iex-ec/about-trust-and-agents-incentives-4651c138974c>`__
- `PoCo series #3: Updated PoCo description <https://medium.com/iex-ec/poco-series-3-poco-protocole-update-a2c8f8f30126>`__

Here is the details of the v3-alpha implementations:

0. **Deal**

   A deal is sealled by the Clerk (TODO: link to the relevant section). This marks the beginning of the execution. An event is produced to notify the worker pool's scheduler.

   The consensus timer starts when the deal is signed. The corresponding task must be finalized before the end of this timer otherwize the scheduler gets punished and the user reimbursed.

1. **Initialization**

   The scheduler calls the ``initialize`` method. Given a dealid and a position in the request order (within the deal window), this function initializes the corresponding task and returns the taskid.

   ``bytes32 taskid = keccak256(abi.encodePacked(_dealid, idx));``

2. **Authorization signature**

   The scheduler designates workers that an participate to this task. This is done by signing (with the scheduler's ethereum wallet) a message containning the worker's ethereum address, the taskid, and (optional) the worker's enclave's ethereum address. If the worker doesn't use an enclave, this field must be filled with ``address(0)``.

   This ethereum signature (authorization) is sent to the worker through an off-chain channel implemented by the middleware.

3. **Task computation**

   Once the authorization received and verified, the worker computes the requested tasks. Results from this execution are placed in the ``/iexec`` folder. The following values are then computed:

   - *bytes32 digest*: a digest (sha256) of the result folder.
   - *bytes32 hash*:   the hash of the *digest*, used to produce a consensus
   - *bytes32 seal*:   the salted hash of the *digest*, used to prove a worker's knew the *digest* value before it is published.

   ``resultHash == keccak256(abi.encodePacked(        taskid, resultDigest))``
   ``resultSeal == keccak256(abi.encodePacked(worker, taskid, resultDigest))``

   An application can overide the computation of the result digest (usually the hash of the result archive) by providing an specific file ``/iexec/consensus.iexec``. This is necessary to achieve consensus on non deterministic applications.

   If an TEE was used to produce the result, the enclave should produce a ``/iexec/enclaveSig.iexec`` that contains the enclave signature (of the resultHash and resultSeal).

   Finally, if the requester asked for a callback, the value of this callback must be specified in ``/iexec/callback.iexec``. Otherwize, the digest will be sent through the callback.

4. **Contribution**

   Once the execution has been performed, the worker has to push its contribution using the ``contribute`` method. The contribution contains:

   - *bytes32 taskid*
   - *bytes32 resultHash*
   - *bytes32 resultSeal*
   - *address enclaveChallenge*

     The address of the enclave (specified in the scheduler's authorization). If no enclave is specified this should be set to ``address(0)``

   - *signature enclaveSign*

     The enclave signature comming from ``/iexec/enclaveSig.iexec``. This is required if the ``enclaveChallenge`` is not ``address(0)``. Otherwize it should be set to ``{ r: bytes32(0), s: bytes32(0), v: uint8(0) }``

   - *signature workerpoolSign*

     The signature computed by the scheduler at step 2.

5. **Consensus**

   During the contribution, the consensus is updated and verified. Contributions are possible until the consensus is reached, at which point the contributions are closed and we enter a 2h reveal phase.

6. **Reveal**

   During the reveal phase, workers that have contributed to the consensus must call the ``reveal`` method with the ``resultDigest``. This verifies that the ``resultHash`` and ``resultSeal`` they provided are valid. Failure to reveal is equivalent to a bad contribution and result in a loss of stake and reputation.

7. **Finalize**

   Once all contribution have been revealed, or at the end of the reveal periode if some (but not all) reveal are missing, the scheduler must call the ``finalize`` method. This finalizes the task, reward good contribution and punish bad ones. This must be called before the end of the consensus timer. If call includes the callback mechanism if it was requested.

Staking and Payment
-------------------

Parameters
----------
