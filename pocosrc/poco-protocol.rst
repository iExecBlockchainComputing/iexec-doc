Protocol
--------

Objectives
~~~~~~~~~~

PoCo stands for proof of contribution. It's protocol designed to provide a trusted computing environment on top of untrusted resources.
In addition to providing trust PoCo also orchestrates the different contributions to the iExec platform so that payment is always fare.

PoCo is modular. It comes with features that can be used depending on the context:

**Result consolidation**

  The PoCo provides a layer of that relies on replication to achieve result consolidation.
  This is a purely software solution that enforces a confidence level on the result. `This confidence level can be customized by the requester <poco-trust.html>`__.

  This layer also support the onchain certification of TEE (such as Intel SGX)

**Secure payment**

  | User funds are locked to ensure all resources providers are paid for their contributions. Resources can take the form of data, applications or computing power.
  | Workers have to achieve consensus to get the user's funds otherwize user is reimbursed.
  | Worker and scheduler have to stake to participate. Bad behaviour from a actor result in a loss of stake.

  This is essential the public blockchain but all values can be set to 0 for private blockchain solutions.

**Permissioning**

  For an execution to happen, a deal must be signed between the different parties. A permission mechanism can be used to control access to applications, datasets and workerpools.
  The secure payment layer can be disabled for a private blockchain.
  It can also be used in the context of the public blockchain to increase the security, with dataset restriction for a specific application for example.


Overview
~~~~~~~~

The PoCo describes the succession of contributions that are required to achieve consensus on a given result. The logic is details in two blog articles:

- `PoCo series #1: Initial PoCo description <https://medium.com/iex-ec/about-trust-and-agents-incentives-4651c138974c>`__
- `PoCo series #3: Updated PoCo description <https://medium.com/iex-ec/poco-series-3-poco-protocole-update-a2c8f8f30126>`__

The `nominal workflow <https://github.com/iExecBlockchainComputing/iexec-doc/raw/master/techreport/nominalworkflow-ODB.png>`__ is also available in the `technical report section <technicalreport.html>`__

Here is the details of the implementations:

0. **Deal**

   `A deal is sealled by the Clerk <poco-brokering.html>`__. This marks the beginning of the execution. An event is produced to notify the worker pool's scheduler.

   The consensus timer starts when the deal is signed. The corresponding task must be finalized before the end of this timer otherwize the scheduler gets punished and the user reimbursed.

1. **Initialization**

   The scheduler calls the ``initialize`` method. Given a deal id and a position in the request order (within the deal window), this function initializes the corresponding task and returns the taskid.

   ``bytes32 taskid = keccak256(abi.encodePacked(_dealid, idx));``

2. **Authorization signature**

   The scheduler designates workers that participate to this task, the scheduler's ethereum wallet signs a message containing the worker's ethereum address, the taskid,
    and (optional) the worker's enclave's ethereum address. If the worker doesn't use an enclave, this field must be filled with ``address(0)``.

   This ethereum signature (authorization) is sent to the worker through an off-chain channel implemented by the middleware.

3. **Task computation**

   Once the authorization received and verified, the worker computes the requested tasks. Results from this execution are placed in the ``/iexec`` folder. The following values are then computed:

   - *bytes32 digest*: a digest (sha256) of the result folder.
   - *bytes32 hash*:   the hash of the *digest*, used to produce a consensus
   - *bytes32 seal*:   the salted hash of the *digest*, used to prove a worker's knew the *digest* value before it is published.

   ``resultHash == keccak256(abi.encodePacked(        taskid, resultDigest))``
   ``resultSeal == keccak256(abi.encodePacked(worker, taskid, resultDigest))``

   An application can override the computation of the result digest (usually the hash of the result archive) by providing an specific file ``/iexec/consensus.iexec``. This is necessary to achieve consensus on non deterministic applications.

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

     The enclave signature coming from ``/iexec/enclaveSig.iexec``. This is required if the ``enclaveChallenge`` is not ``address(0)``. Otherwise it should be set to ``{ r: bytes32(0), s: bytes32(0), v: uint8(0) }``

   - *signature workerpoolSign*

     The signature computed by the scheduler at step 2.

5. **Consensus**

   During the contribution, the consensus is updated and verified. Contributions are possible until the consensus is reached,
    at which point the contributions are closed and we enter a 2h reveal phase.

6. **Reveal**

   During the reveal phase, workers that have contributed to the consensus must call the ``reveal`` method with the ``resultDigest``. This verifies that the ``resultHash`` and ``resultSeal`` they provided are valid.
    Failure to reveal is equivalent to a bad contribution and result in a loss of stake and reputation.

7. **Finalize**

   Once all contribution have been revealed, or at the end of the reveal period if some (but not all) reveals are missing, the scheduler must call the ``finalize`` method.
    This finalizes the task, reward good contribution and punish bad ones. This must be called before the end of the consensus timer. If call includes the callback mechanism if it was requested.

Staking and Payment
~~~~~~~~~~~~~~~~~~~

Among the objectives of PoCo, we want to ensure a worker that contributes correctly is rewarded and, at the same time, that a requester won't be changed unless a consensus is achieved.
 This is achieved by locking the requesters funds for the duration of the consensus, and unlocking them depending on the outcomes.

Workers have to stake to prevent bad behaviour and ensure only good contributions are viable.

Your iExec account, managed by the ``Escrow`` part of the ``IexecClerk``, separates between ``balance.stake`` (available, can be withdrawn) and ``balanced.locked`` (unavailable, frozen by a running task).
The ``Escrow`` exposes the following mechanism:

``lock``: Moves value from the ``balance.stake`` to ``balance.lock``

  - Locks the requester stake for payment
  - Locks the scheduler stake to protect against failed consensus
  - Locks the worker stake when making a contribution

``unlock``: Moves value from the ``balance.lock`` back to the ``balance.stake``

  - Unlock the requester stake when the consensus fails
  - Unlock the scheduler stake when consensus is achieved
  - Unlock the worker stake when they contributed to a successfull consensus

``seize``: Confiscate value from ``balance.lock``

  - Seize the requester stake when the consensus is achieved (payment)
  - Seize the scheduler stake when consensus fails (send to the reward kitty)
  - Seize the worker stake when a contribution fails (redistributed to the other workers in the task)

``reward``: Award value to the ``balance.stake``

  - Reward the scheduler when consensus is achieved
  - Reward the worker when they contributed to a successfull consensus
  - Reward the app and dataset owner

The requester payment is composed of 3 parts, one for the workerpool, one for the application and one for the dataset.
 When a consensus is finalized, the payment is seized from the requester and the application and dataset owners are rewarded accordingly.
 The workerpool part is put inside the ``totalReward``. Stake from the losing workers is also added to the ``totalReward``.
 The scheduler takes a fixed portion of the ``totalReward`` as defined in the workerpool smartcontract (``schedulerRewardRatioPolicy``).
 The remaining reward is then divided between the successfull workers proportionnaly to the impact their contribution made on the consensus.
 If there is anything left (division rounding, a few nRLC at most) the scheduler gets is. The scheduler also gets part of the reward kitty.

Parameters
~~~~~~~~~~

``FINAL_DEADLINE_RATIO = 10``, ``CONTRIBUTION_DEADLINE_RATIO = 7``, ``REVEAL_DEADLINE_RATIO = 2``

  Parameters of the consensus timer. They express the number of reference timers (category duration) that are dedicated to each phase.
  These settings corresponds to a 70%-20%-10% distribution between the contribution phase, the reveal phase and the finalize phase.

    - ``FINAL_DEADLINE_RATIO`` This describes the total duration of the consensus. At the end of this timer the consensus must be finalized. If it is not, the user can make a claim to get a refund.

    - ``CONTRIBUTION_DEADLINE_RATIO`` This describes the duration of the contribution period. The consensus can finalize before that, but no contribution will be allowed after the timer to ensure enough time is left for the reveal and finalize steps.

    - ``REVEAL_DEADLINE_RATIO`` This describes the duration of the reveal period. Whenever a contribution triggers a consensus, a reveal period of this duration is reserved for the workers to reveal their contribution.
     Note that this period will necessarily start before the end of the contribution phase.

  Lets consider a task of category `GigaPlus`, which reference duration is 1 hour. If the task was submitted at 9:27AM, the contributions must be sent before 4:27PM (16:27).
   Whenever a contribution triggers a consensus, a 2 hours long reveal period will start. Whatever happens, the consensus has to been achieved by 7:27PM (19:27).

``WORKERPOOL_STAKE_RATIO = 30``

  Percentage of the workerpool price that has to be staked by the scheduler. For example, for a ``20 RLC`` task, with an additional ``1 RLC`` for the application and ``5 RLC`` for the dataset, the worker will have to lock ``26 RLC`` in total and the scheduler will have to lock (stake) ``30% * 20 = 6 RLC``.

  This stake is lost and transferred to the reward kitty if the consensus is not finalized by the end of the consensus timer.

``KITTY_RATIO = 10``

  Percentage of the reward kitty for the scheduler per successful execution. If the reward kitty contains 42 RLC when a finalize is called,
   then the scheduler will get 4.2 extra RLC and the reward kitty will be left with 37.8 RLC.

``KITTY_MIN = 1 RLC``

  Minimum reward on successful execution (up to the reward kitty value).

  - If the reward kitty contains 42.0 RLC, the reward is 4.2
  - If the reward kitty contains 5.0 RLC, the reward should be 0.5 but gets raised to 1.0
  - If the reward kitty contains 0.7 RLC, the reward should be 0.07 but gets raised to 0.7 (the whole kitty)

  ``reward = kitty.percentage(KITTY_RATIO).max(KITTY_MIN).min(kitty)``

Example
~~~~~~~

Lets consider a workerpool with the policies ``workerStakeRatioPolicy = 35%`` and ``workerStakeRatioPolicy = 5%``.

- A requester offers ``20 RLC`` to run a task. The task is free but it uses a dataset that cost ``1 RLC``. The requester locks ``21 RLC`` and the scheduler ``30% * 20 = 6 RLC``. The trust objective is ``99%`` (``trust = 100``)

- 3 workers contribute:

  - The first one (``score = 12 → power = 3``) contributes ``17``. He has to lock ``7 RLC`` (35% of the ``20 RLC`` awarded to the worker pool).
  - The second worker (``score = 100 → power = 32``) contributes ``42``. He also locks ``7 RLC``.
  - The third worker (``score = 300 → power = 99``) contributes ``42``. He also locks ``7 RLC``.

- After the third contribution, the value ``42`` has reached a ``99.87%`` likelihood. Consensus is achieved and the two workers who contributed toward ``42`` have to reveal.

- After both workers reveal, the scheduler finalizes the task:

  - The requester locked value of ``21 RLC`` is seized.
  - The dataset owner gets ``1 RLC`` for the use of its dataset.
  - Stake from the scheduler is unlocked.
  - Stakes from workers 2 and 3 are also unlocked.
  - The first workers stake is seized and he loses a third of its score. The correspond ``7 RLC`` are added to the ``totalReward``
  - We now have ``totalReward = 27 RLC``:

    - We save 5% for the scheduler, ``workersReward = 95% * 27 = 25.65 RLC``
    - Worker 2 has weight ``log2(32) = 5`` and worker 3 has a weight ``log2(99) = 6``. Total weight is ``5+6=11``
    - Worker 2 takes ``25.65 * 5/11 = 11.659090909 RLC``
    - Worker 3 takes ``25.65 * 6/11 = 13.990909090 RLC``
    - Scheduler takes the remaining ``1.350000001 RLC``

  - If the reward kitty is not empty, the scheduler also takes a part of it.
