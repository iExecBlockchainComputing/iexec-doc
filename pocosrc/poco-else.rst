Other technical choices
-----------------------

Callback
~~~~~~~~

Some requester might want an onchain callback with the result of the execution. The callback mechanism is based on [EIP1154]_.
The result is a ``bytes`` value that is set during the ``finalize``. The ``IexecHub`` implements both side of the [EIP1154]_.

**Pull**

  Results are identified by their ``taskid`` and can be pulled through the ``resultFor`` method.

**Push**

  In order to use the push approach, the requester can use the ``callback`` field to specify the address of a smart contract that implement the ``receiveResult`` method specified in [EIP1154]_.
  This method will be called during the finalization with at minimum of 100000 nanoRLC gas to proceed [*]_.

  In order to protect the scheduler and the workers, any error raised during this callback will be disregarded and will not prevent the finalization from happening. The same mechanism goes for the callback running out of gas.

Consensus & Reveal duration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

When orders match, IexecClerk records the deal which details the parameters of the task. If a consensus on a result is achieved before the countdown, the task is successful.
 After the countdown, as no consensus is reached, the execution is failed.
 The duration of the consensus timer is a balance between the quality of service offered to the requester (short timer) and the margin available for the scheduler and the worker to achieve consensus (and go through the reveal process).

The maximum duration of a task is governed by the category the task fits in. While the consensus duration can obviously not be shorter than the task runtime a significant margin is required for the scheduler to do its job correctly. Multiple workers are likely to contribute and extra time must be planed for the revealing and finalization steps.

The consensus timer starts when the deal is recorded by the IexecClerk.

In case of failure, the requester can claim the refund.

In the v3-alpha version, this timer lasts for 10 category runtime, for all category.

| The reveal timer starts whenever a consensus is reached and determines the time frame the workers have to reveal their contributions.
| This should be long enough for worker to have time to reveal while not being to long so that the requesters waits to long for its result or the consensus fails because the scheduler cannot finalize in time.
 In the v3-alpha version, this timer lasts for 2 runtime whatever the category runtime. This leaves a gap of at least 1 time the category runtime for the scheduler to finalize the task.


Reward kitty
~~~~~~~~~~~~

| When a consensus fails, the requester gets a refund and the scheduler loses its stake. In order to remove an attack vector, the requester does not get any of the seized stake.
| If this was a feature, anyone could build a flawed application that would not reach consensus to drain money from the scheduler.
| This would force the scheduler to whitelist all applications thus reducing the usability of the platform.

| Seized stake from the scheduler goes into a specific account called the *reward kitty*. No one controls this account, nor can withdraw from it.
| However, the tokens are not burned. Whenever a task is finalized, the scheduler that organized the execution of this task is rewarded by the requester and also gets a small part of the reward kitty.

As described in the `protocol parameters <poco-protocol.html#parameters>`_ section, this reward is ``reward = kitty.percentage(KITTY_RATIO).max(KITTY_MIN).min(kitty)``.


References
~~~~~~~~~~

.. [EIP1154] `EIP 1154: Oracle Interface <https://eips.ethereum.org/EIPS/eip-1154>`_
.. [*] value susceptible to change.
