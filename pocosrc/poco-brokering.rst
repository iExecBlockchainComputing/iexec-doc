Brokering
---------

Overview
~~~~~~~~

For iExec v3 we studied many possible evolutions of the brokering process. The first requirement was to include bid orders in addition to the already available ask orders. It soon became clear that the evolution had to go beyond a simple interaction. We considered on-chain and off-chain approach and finally went for a solution where both the pairing and the order book are off-chain. It might sound counterintuitive, but it has many advantages.

If the orders and the pairing are handled off-chain, how can we build an on-chain agreement knowing that all parties have agreed? Is there a threat to the platform security?

This option relies on the use of cryptographic signatures for order authentication. We represent orders using structures containing all the required details. The hash of this structure uniquely identifies the order. The structure by itself is worthless as anyone could write and publish it. However, if we add a valid cryptographic signature (of the identifying hash), then the origin of the order can be certified with the same level of security as if it was published on-chain. This role is fulfilled by a smart-contract called the iExecClerk.

An overiew of the iExecODB (Open Decentralized Brokering) is available in this blog article:

- `PoCo series #5: Open decentralized brokering on the iExec platform <https://medium.com/iex-ec/poco-series-5-open-decentralized-brokering-on-the-iexec-platform-67b266e330d8>`__

Orders structure
~~~~~~~~~~~~~~~~

As discussed earlier, a core element of the iExec Open Decentralized Brokering introduced in version 3 is the offchain signature of orders that are then matched by the iExec Clerk. There are 4 types of orders corresponding to the 4 actors that can take part in execution: The workerpool, the application, the dataset and the requester. Each order types has to follow a specific structure and is signed using `EIP712`_ structure signature mechanism.

The structure of the different order types is as follows:

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

- ``datasetrestrict`` Matching restrictions. Dataset or group of datasets that can be matched. Let null value to disable check.

- ``workerpoolrestrict`` Matching restrictions. Workerpool or group of workerpools that can be matched. Let null value to disable check.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Let null value to disable check.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the application owner specified in the app smartcontract for the order to be valid.

**DatasetOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 43-54
  :dedent: 1

- ``dataset`` Address of the smartcontract describing the dataset. Must be registered in the DatasetRegistry.

- ``datasetprice`` Price of a use of the dataset.

- ``volume`` Number of authorized uses (by this order).

- ``tag`` Special requirements of the dataset (see `tag`_).

- ``apprestrict`` Matching restrictions. App or group of apps that can be matched. Let null value to disable check.

- ``workerpoolrestrict`` Matching restrictions. Workerpool or group of workerpools that can be matched. Let null value to disable check.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Let null value to disable check.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the dataset owner specified in the dataset smartcontract for the order to be valid.

**WorkerpoolOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 55-68
  :dedent: 1

- ``workerpool`` Address of the smartcontract describing the workerpool. Must be registered in the WorkerpoolRegistry.

- ``workerpoolprice`` Price of a one execution on the workerpool.

- ``volume`` Number of executions proposed (by this order).

- ``tag`` Special features proposed by the workerpool (see `tag`_).

- ``category`` Category proposed by this order.

- ``trust`` Trust level used to certify results.

- ``apprestrict`` Matching restrictions. App or group of apps that can be matched. Let null value to disable check.

- ``datasetrestrict`` Matching restrictions. Dataset or group of datasets that can be matched. Let null value to disable check.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Let null value to disable check.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the scheduler specified in the workerpool smartcontract for the order to be valid.

**RequesterOrder**

.. literalinclude:: IexecODBLibOrders.sol
  :language: c
  :lines: 69-87
  :dedent: 1

- ``app`` Address of the smartcontract describing the application. Must be registered in the AppRegistry.

- ``appmaxprice`` Maximum price allowed by the requester for the payment of the application.

- ``dataset`` Address of the smartcontract describing the dataset. Must be registered in the DatasetRegistry. Leave empty (null) if no dataset is required

- ``datasetmaxprice`` Maximum price allowed by the requester for the payment of the dataset (if any).

- ``workerpool`` Matching restrictions. Workerpool or group of workerpools that are allowed to run tasks from this order. Leave null value to disable check.

- ``workerpoolmaxprice`` Maximum price allowed by the requester for the payment of the execution (scheduler + workers).

- ``requester`` Address of the requester (paying for the executions).

- ``volume`` Number of tasks that are part of this order (size of the Bag Of Task).

- ``tag`` Special features required by the requester (see `tag`_).

- ``category`` Category required for the execution of the tasks in this order.

- ``trust`` Minimum trust level required by the requester.

- ``beneficiary`` Address of the beneficiary of the computation. Used to require output data encryption.

- ``callback`` Address to callback with the results (following EIP1154 interface). Let empty (null) if no callback is required. Learn more about the `callback mechanism`_.

- ``params`` Parameters of the application (application specific).

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the requester for the order to be valid.

.. _callback mechanism: poco-else.html#callback

Tag
---

.. _tag: #tag

The tag is used to specify the need or the availability of features that go beyound the specifications of the category. The tag is a requierement when it is part of an app order, a dataset order or a requester order. On the other hand, the tag in the workerpoolorder express the availability of the corresponding features.

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

For orders to match, the workerpool order must enable all bits that are enables in any of the app order, dataset order and requester order. Meaning that if the app order tag is ``0x12 = 0x10 | 0x02``, the dataset order is ``0x81 = 0x80 | 0x01`` and the requester order is ``0x03 = 0x02 | 0x01``, than the workerpool order must, at least, have a tag ``0x93 = 0x80 | 0x10 | 0x02 | 0x01``.

Matching Conditions
~~~~~~~~~~~~~~~~~~~

In order to trigger an execution, a deal must be registered by the iExec Clerk. Deals are produced when orders are succesfully matched by the clerk. A matching requires 3 or 4 orders depending on the requester requierements (the dataset order can be left empty if the execution does not involve a dataset). The order matched must all be compatible with one another:

1.
	The workerpool's category and the requester's category must be equal.

	``require(_requestorder.category == _workerpoolorder.category);``

2.
	The workerpool's trust must be greater or equal to the requester's trust.

	``require(_requestorder.trust == _workerpoolorder.trust);``

3.
	The app's price, dataset's price and workerpool's price must be less or equal to the requester's appmaxprice, datasetmaxprice and workerpoolmaxprice.

	``require(_requestorder.appmaxprice >= _apporder.appprice);``
	``require(_requestorder.datasetmaxprice >= _datasetorder.datasetprice);``
	``require(_requestorder.workerpoolmaxprice >= _workerpoolorder.workerpoolprice);``

4.
	The workerpool's tag must enable all the features required by the app's tag, the dataset's tag and the workerpool's tag.

	``require((_apporder.tag | _datasetorder.tag | _requestorder.tag) & ~_workerpoolorder.tag == 0x0);``

5.
	The app provided by the apporder must match the one required by the requester.

	``require(_requestorder.app == _apporder.app);``

6.
	The dataset provided by the datasetorder must match the one required by the requester.

	``require(_requestorder.dataset == _datasetorder.dataset);``

7.
	If the requester specified a workerpool restriction, the workerpool must match this value or be part of the corresponding group.

	``require(checkRestriction(_requestorder.workerpool, _workerpoolorder.workerpool, 0x01));``

8.
	The application must fit the dataset's and the workerpool's application restrictions (if any).

	``require(checkRestriction(_datasetorder.apprestrict, _apporder.app, 0x01));``
	``require(checkRestriction(_workerpoolorder.apprestrict, _apporder.app, 0x01));``

9.
	The dataset must fit the application's and the workerpool's restrictions (if any).

	``require(checkRestriction(_apporder.datasetrestrict, _datasetorder.dataset, 0x01));``
	``require(checkRestriction(_workerpoolorder.datasetrestrict, _datasetorder.dataset, 0x01));``

10.
	The workerpool must fit the application's and the dataset's restrictions (if any).

	``require(checkRestriction(_apporder.workerpoolrestrict, _workerpoolorder.workerpool, 0x01));``
	``require(checkRestriction(_datasetorder.workerpoolrestrict, _workerpoolorder.workerpool, 0x01));``

11.
	The requester must fit the application's, the dataset's and the workerpool's restrictions (if any).

	``require(checkRestriction(_apporder.requesterrestrict, _requestorder.requester, 0x01));``
	``require(checkRestriction(_datasetorder.requesterrestrict, _requestorder.requester, 0x01));``
	``require(checkRestriction(_workerpoolorder.requesterrestrict, _requestorder.requester, 0x01));``

12.
	The application, dataset and workerpool must be registered in the iExecHub.

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
	Requester and Scheduler must be able to stake.

FAQ : How to write an order ?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**[Requester] How do I enable PoCo's certification of results?**

A requester can enable the trust layer of the PoCo by setting the trust value in que requestorder. As described `here <poco-trust.html>`__, the trust can be customized depending on the required confidence level. If the requester wants à 99.99% confidence level on the results, it must set the ``trust`` field to ``10000``

**[Requester] How do I run a non determinist application despite the PoCo requiering determinism?**

The PoCo requires an application to be deterministic for the replication layer to provide trust in the result. If an application is not deterministic, consensus cannot be achieved and replication must be avoided.

In order to obtain a result, the requester must prevent replication by asking a ``trust`` value of ``0``. If the requester want to get some result protection, and if the application is compatible with it, the requester can the task to run in an enclave by setting the ``tag`` to ``0x1``.

**[Requester] How do I protect my result using encryption?**

The result of an execution can be valuable to the end user, and the requester might want to protect this result from leaking by encrypting it.

Anyone can set up an encryption key in a SMS (Secret Management Service) of its choice and set up the SMS address in the directory.

Once a user as set an encryption key (see TODO), any computation result can be encrypted with it by simply setting up the beneficiary address in the RequesterOrder.

An application can only perform result encryption if it runs in a verifyed enclave. No encryption key will be provided by the SMS to an application that doesn't run in a certified enclave.

(TODO: potential issue, key leaking to malicious application with the requester attacking a beneficiary)

**[Dataset owner] How do I limit the usage of my dataset to a specific application**

iExec's datawallet and datastore provide all the features to monetize a valuable dataset while protecting it. Before uploading a dataset you should encrypt it using the iExec SDK. Through this process, the encryption key becomes the valuable data that has to be protected.

The encryption key must be stored in a SMS and the address of the corresponding SMS recorded in the directory. The SMS stores this encryption key and will only communicate it to an application running in an enclave.

For a worker running an application to get the encryption key, it must first prove that this access is legitimate by providing the scheduler authorization. The SMS will verify that this authorization's signature is valid and that the corresponding task is registered onchain. This means that any deal signed in the iExec Clerk will grant access to the dataset's encryption key.

Therefore in order to restrict the dataset's usage, the dataset owner should specify everything before hand when signing a brokering order. This is done through the ``apprestrict`` field of the datasetorder. The datasetowner can deploy a ``SimpleGroup`` smart contract, have the ``apprestrict`` field point to it, and then whitelist the applications that will have access to the dataset's encryption key.

**[Scheduler] How do I protect myself from non deterministic applications?**

When a scheduler publishes an order, it makes a commitment to achieve consensus on any task that is part of a deal made using this order. While everything is done to ensure an application cannot hurt a worker pool's machines, not reaching the consensus would cause a loss of stake from the scheduler. The scheduler must therefore take action to prevent this.

Whenever the scheduler proposes to certify a result using the PoCo's trust layer, it should make sure the application's developper took the actions required to avec it compatible with the PoCo. On the other hand, any application could be executed with the PoCo's trust layer disabled.

A scheduler could therefore emit two kinds of workerpoolorder:

- A workerpoolorder offering execution with the PoCo's trust layer disabled (``trust = 0``) and accepting all applications (``apprestrict = 0``)

- A workerpoolorder offering secure exection of whitelisted tasks. The application whitelist would use the ``GroupInterface`` to be verifyed by the iExec Clerk. This group could either be managed by the scheduler or by a certification authority that would check applications determinism.
