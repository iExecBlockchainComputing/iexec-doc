Brokering
---------

The blockering will be released in june 2019, with the version 3 of the platform

Overview
~~~~~~~~

We studied many possible evolutions of the brokering process. The first requirement is to include bid orders to complete the already available ask orders. It soon became clear that the evolution had to go beyond a simple interaction. We considered on-chain and off-chain approach and finally went for a solution where both the pairing and the order book are off-chain. It might sound counterintuitive, but it has many advantages.

If the orders and the pairing are handled off-chain, how can we build an on-chain agreement knowing that all parties have agreed? Is there a threat to the platform security?

This option relies on the use of cryptographic signatures for order authentication. We represent orders using structures containing all the required details. The hash of this structure uniquely identifies the order. The structure by itself is worthless as anyone could write and publish it. However, if we add a valid cryptographic signature (of the identifying hash), then the origin of the order can be certified with the same level of security as if it was published on-chain. This role is fulfilled by a smart-contract called the iExecClerk.

An overview of the iExecODB (Open Decentralized Brokering) is available in this blog article:

- `PoCo series #5: Open decentralized brokering on the iExec platform <https://medium.com/iex-ec/poco-series-5-open-decentralized-brokering-on-the-iexec-platform-67b266e330d8>`__

Orders structure
~~~~~~~~~~~~~~~~

As discussed earlier, iExec introduces the offchain signature of orders as a new core element of the iExec Open Decentralized Brokering, the iExec Clerk should match these orders.
There are 4 types of orders corresponding to the 4 actors involved: the worker pool, the application, the dataset and the requester.
Each order types has to follow a specific structure and is signed using `EIP712`_ structure signature mechanism.


Orders description:
~~~~~~~~~~~~~~~~~~~

.. _EIP712: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md

**AppOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 31-42
  :dedent: 1

- ``app`` Address of the smartcontract describing the application. Must be registered in the AppRegistry.

- ``appprice`` Price of a run of the application.

- ``volume`` Number of run authorized (by this order).

- ``tag`` Special requirements for the application (see `tag`_).

- ``datasetrestrict`` Matching restrictions. Dataset or group of datasets that can be matched. Let null value to disable.

- ``workerpoolrestrict`` Matching restrictions. Worke rpool or group of worker pools that can be matched. Let null value to disable.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Let null value to disable.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order, the smart contract is securely linked to application owner.

**DatasetOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 43-54
  :dedent: 1

- ``dataset`` Address of the smartcontract describing the dataset. Must be registered in the DatasetRegistry.

- ``datasetprice`` Price of a use of the dataset.

- ``volume`` Number of authorized uses (by this order).

- ``tag`` Special requirements of the dataset (see `tag`_).

- ``apprestrict`` Matching restrictions. App or group of apps that can be matched. Let null value to disable.

- ``workerpoolrestrict`` Matching restrictions. Workerpool or group of workerpools that can be matched. Let null value to disable.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Let null value to disable.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order, the smart contract is securely linked to dataset owner.

**WorkerpoolOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 55-68
  :dedent: 1

- ``workerpool`` Address of the smartcontract describing the worker pool. Must be registered in the WorkerpoolRegistry.

- ``workerpoolprice`` Price of an execution on the worker pool.

- ``volume`` Number of executions proposed (by this order).

- ``tag`` Special features proposed by the worke rpool (see `tag`_).

- ``category`` Order category.

- ``trust`` Trust level used to consolidated results.

- ``apprestrict`` Matching restrictions. App or group of apps that can be matched. Let null value to disable.

- ``datasetrestrict`` Matching restrictions. Dataset or group of datasets that can be matched. Let null value to disable.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Let null value to disable.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order, the smart contract is securely linked to worker pool manager.

**RequesterOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 69-87
  :dedent: 1

- ``app`` Address of the smartcontract describing the application. Must be registered in the AppRegistry.

- ``appmaxprice`` Maximum price allowed by the requester for the payment of the application.

- ``dataset`` Address of the smartcontract describing the dataset. Must be registered in the DatasetRegistry. Null if no dataset is required.

- ``datasetmaxprice`` Maximum price allowed by the requester for the payment of the dataset (if any).

- ``workerpool`` Matching restrictions. Worker pool or group of worker pools that are allowed to run tasks from this order. Leave null value to disable check.

- ``workerpoolmaxprice`` Maximum price allowed by the requester for the payment of the execution (scheduler + workers).

- ``requester`` Address of the requester (paying for the executions).

- ``volume`` Number of tasks that are part of this order (size of the Bag Of Task).

- ``tag`` Special features required by the requester (see `tag`_).

- ``category`` tasks category required.

- ``trust`` Minimum trust level required by the requester.

- ``beneficiary`` Address of the beneficiary of the computation. Used to require output data encryption.

- ``callback`` Address to callback with the results (following EIP1154 interface). Let empty (null) if no callback is required. Learn more about the `callback mechanism`_.

- ``params`` Parameters of the application (application specific).

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order, the smart contract is securely linked to requester.

.. _callback mechanism: poco-else.html#callback

Tag
---

.. _tag: #tag

The tag specifies the need or the availability of features that go beyond the specifications of the category. The tag is a requirement when it is part of an app order, a dataset order or a requester order.
On the other hand, the tag in the workerpoolorder express the availability of the corresponding features.

In V3, tags are 32 bytes (256 bits) long array where each bit corresponds to a feature.

+------------------------------------------------------------------------+------------------------+
| **Value**                                                              | **Description**        |
+------------------------------------------------------------------------+------------------------+
| ``0x0000000000000000000000000000000000000000000000000000000000000001`` | TEE (physical enclave) |
+------------------------------------------------------------------------+------------------------+
| ``0x0000000000000000000000000000000000000000000000000000000000000002`` | —                      |
+------------------------------------------------------------------------+------------------------+
| ``0x0000000000000000000000000000000000000000000000000000000000000004`` | —                      |
+------------------------------------------------------------------------+------------------------+
| ``0x0000000000000000000000000000000000000000000000000000000000000008`` | —                      |
+------------------------------------------------------------------------+------------------------+
| ``0x0000000000000000000000000000000000000000000000000000000000000010`` | —                      |
+------------------------------------------------------------------------+------------------------+
| ...                                                                    | ...                    |
+------------------------------------------------------------------------+------------------------+
| ``0x8000000000000000000000000000000000000000000000000000000000000000`` | —                      |
+------------------------------------------------------------------------+------------------------+

For orders matching, the worker pool order must enable all bits that are enables in any of the app order, dataset order and requester order. Meaning that if the app order tag is ``0x12 = 0x10 | 0x02``,
the dataset order is ``0x81 = 0x80 | 0x01`` and the requester order is ``0x03 = 0x02 | 0x01``, then the worker pool order must, at least, have a tag ``0x93 = 0x80 | 0x10 | 0x02 | 0x01``.

Matching Conditions
~~~~~~~~~~~~~~~~~~~

In order to trigger an execution, a deal must be registered by the iExec Clerk. Deals are produced when orders are succesfuly matched by the clerk.
A matching requires 3 or 4 orders depending on the requester requirements, the dataset order is optional.

Orders compatibility required:

1.
	The worker pool's category and the requester's category must be equal.

	``require(_requestorder.category == _workerpoolorder.category);``

2.
	The worker pool's trust must be greater or equal to the requester's trust.

	``require(_requestorder.trust == _workerpoolorder.trust);``

3.
	The app's, dataset's and worker pool's prices must be less or equal to the requester's appmaxprice, datasetmaxprice and workerpoolmaxprice.

	``require(_requestorder.appmaxprice >= _apporder.appprice);``
	``require(_requestorder.datasetmaxprice >= _datasetorder.datasetprice);``
	``require(_requestorder.workerpoolmaxprice >= _workerpoolorder.workerpoolprice);``

4.
	The worker pool's tag must enable all the features required by the app's tag, the dataset's tag and the worker pool's tag.

	``require((_apporder.tag | _datasetorder.tag | _requestorder.tag) & ~_workerpoolorder.tag == 0x0);``

5.
	The app provided by the apporder must match the one required by the requester.

	``require(_requestorder.app == _apporder.app);``

6.
	The dataset provided by the datasetorder must match the one required by the requester.

	``require(_requestorder.dataset == _datasetorder.dataset);``

7.
	If the requester specified a worker pool restriction, the worker pool must match this value or be part of the corresponding group.

	``require(checkRestriction(_requestorder.workerpool, _workerpoolorder.workerpool, 0x01));``

8.
	The application must fit the dataset's and the worker pool's application restrictions (if any).

	``require(checkRestriction(_datasetorder.apprestrict, _apporder.app, 0x01));``
	``require(checkRestriction(_workerpoolorder.apprestrict, _apporder.app, 0x01));``

9.
	The dataset must fit the application's and the worker pool's restrictions (if any).

	``require(checkRestriction(_apporder.datasetrestrict, _datasetorder.dataset, 0x01));``
	``require(checkRestriction(_workerpoolorder.datasetrestrict, _datasetorder.dataset, 0x01));``

10.
	The worker pool must fit the application's and the dataset's restrictions (if any).

	``require(checkRestriction(_apporder.workerpoolrestrict, _workerpoolorder.workerpool, 0x01));``
	``require(checkRestriction(_datasetorder.workerpoolrestrict, _workerpoolorder.workerpool, 0x01));``

11.
	The requester must fit the application's, the dataset's and the worker pool's restrictions (if any).

	``require(checkRestriction(_apporder.requesterrestrict, _requestorder.requester, 0x01));``
	``require(checkRestriction(_datasetorder.requesterrestrict, _requestorder.requester, 0x01));``
	``require(checkRestriction(_workerpoolorder.requesterrestrict, _requestorder.requester, 0x01));``

12.
	The application, dataset and worker pool must be registered in the iExecHub.

	``require(iexechub.checkResources(_apporder.app, _datasetorder.dataset, _workerpoolorder.workerpool));``

13.
	All orders must be signed or presigned.

	``require(verify(App(_apporder.app).m_owner(), _apporder.hash(), _apporder.sign));``
	``require(verify(Dataset(_datasetorder.dataset).m_owner(), _datasetorder.hash(), _datasetorder.sign));``
	``require(verify(Workerpool(_workerpoolorder.workerpool).m_owner(), _workerpoolorder.hash(), _workerpoolorder.sign));``
	``require(verify(_requestorder.requester, _requestorder.hash(), _requestorder.sign));``

14.
	The deal produced must contain at least one task.

15.
	Requester and worker pool must be able to stake.

FAQ : How to write an order ?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**[Requester] How do I enable PoCo's consolidation of results?**

A requester can enable the trust layer of the PoCo by setting the trust value in tue requestorder. As described `here <poco-trust.html>`__, the trust is defined with the required confidence level. If the requester wants à 99.99% confidence level on the results, it must set the ``trust`` field to ``10000``

**[Requester] How do I run a non deterministic application despite requiring determinism?**

The PoCo requires an application to be deterministic for the replication layer to provide trust in the result. If an application is not deterministic, consensus cannot be achieved and replication is not necessary.

In order to obtain a result, the requester must prevent replication by asking a ``trust`` value of ``0``.
To protect your result, the requester can ask to run in an enclave by setting the ``tag`` to ``0x1``.

**[Requester] How do I protect my result using encryption?**

The result of an execution can be valuable to the end user, and the requester might want to protect this result from leaking with encryption.

Anyone can set up an encryption key in a SMS (Secret Management Service) of its choice and set up the SMS address in the directory.

Once a user set an encryption key (see TODO), any computation result can be encrypted with, you have to set up the beneficiary address in the RequesterOrder.

An application can only perform result encryption inside an enclave. No encryption key will be provided by the SMS to an application that doesn't run outside an enclave.

(TODO: potential issue, key leaking to malicious application with the requester attacking a beneficiary)

**[Dataset owner] How do I limit the usage of my dataset to a specific application**

iExec's Data wallet and Data store is a complete solution to monetize valuable datasets preserving privacy.
Before uploading a dataset you should encrypt it using the iExec SDK. Through this process, the encryption key becomes the valuable data that has to be protected.

The encryption key must be stored in a SMS and the address of the corresponding SMS recorded in the directory. The SMS stores this encryption key and will only communicate it to an application running in an enclave.

Before a worker runs this application, the worker must first prove that its access is legitimate by providing the scheduler authorization. The SMS will verify that this authorization's signature is valid and that the corresponding task is registered onchain. This means that any deal signed in the iExec Clerk will grant access to the dataset's encryption key.

Therefore in order to restrict the dataset's usage, the dataset owner should et up restriction before signing a brokering order. This is done through the ``apprestrict`` field of the datasetorder.
The dataset owner can deploy a ``SimpleGroup`` smart contract, have the ``apprestrict`` field point to it, and then whitelisting the applications that will have access to the dataset's encryption key.

**[Scheduler] How do I protect myself from non deterministic applications?**

When a scheduler publishes an order, it makes a commitment to achieve consensus on any task that is part of a deal made.
While everything is done to ensure an application cannot hurt a worker pool's machines, not reaching the consensus would cause a loss of stake for the scheduler. The scheduler must therefore take action to prevent this.

Whenever the scheduler proposes to certify a result using the PoCo's trust layer, it should make sure the application's developper took the actions required to avec it compatible with the PoCo. On the other hand, any application could be executed with the PoCo's trust layer disabled.

A scheduler could therefore emit two kinds of workerpoolorder:

- A workerpoolorder offering execution with the PoCo's trust layer disabled (``trust = 0``) and accepting all applications (``apprestrict = 0``)

- A workerpoolorder offering secure exection of whitelisted tasks. The application whitelist would use the ``GroupInterface`` to be verifyed by the iExec Clerk. This group could either be managed by the scheduler or by a certification authority that would check applications determinism.
