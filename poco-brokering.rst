Brokering
=========

Overview
--------

... TODO ...

Orders structure
----------------

As discussed earlier, a core element of the iExec Open Decentralized Brokering introduced in version 3 is the offchain signature of orders that are then matched by the iExec Clerk. There are 4 types of orders corresponding to the 4 actors that can take part in execution: The workerpool, the application, the dataset and the requester. Each order types has to follow a specific structure and is signed using `EIP712`_ structure signature mechanism.

The structure of the different order types is as follows:

.. _EIP712: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md

**AppOrder**

.. literalinclude:: sources/IexecODBLibOrders.sol
  :language: c
  :lines: 31-42
  :dedent: 1

- ``app`` Address of the smartcontract describing the application. Must be registered in the AppRegistry.

- ``appprice`` Price of a run of the application.

- ``volume`` Number of run authorized (by this order).

- ``tag`` Special requierements of the application (see `tag`_).

- ``datasetrestrict`` Matching restrictions. Dataset or group of datasets that can be matched. Leave null value to disable check.

- ``workerpoolrestrict`` Matching restrictions. Workerpool or group of workerpools that can be matched. Leave null value to disable check.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Leave null value to disable check.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the application owner specified in the app smartcontract for the order to be valid.

**DatasetOrder**

.. literalinclude:: sources/IexecODBLibOrders.sol
  :language: c
  :lines: 43-54
  :dedent: 1

- ``dataset`` Address of the smartcontract describing the dataset. Must be registered in the DatasetRegistry.

- ``datasetprice`` Price of a use of the dataset.

- ``volume`` Number of authorized uses (by this order).

- ``tag`` Special requierements of the dataset (see `tag`_).

- ``apprestrict`` Matching restrictions. App or group of apps that can be matched. Leave null value to disable check.

- ``workerpoolrestrict`` Matching restrictions. Workerpool or group of workerpools that can be matched. Leave null value to disable check.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Leave null value to disable check.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the dataset owner specified in the dataset smartcontract for the order to be valid.

**WorkerpoolOrder**

.. literalinclude:: sources/IexecODBLibOrders.sol
  :language: c
  :lines: 55-68
  :dedent: 1

- ``workerpool`` Address of the smartcontract describing the workerpool. Must be registered in the WorkerpoolRegistry.

- ``workerpoolprice`` Price of a one execution on the workerpool.

- ``volume`` Number of executions proposed (by this order).

- ``tag`` Special features proposed by the workerpool (see `tag`_).

- ``category`` Category proposed by this order.

- ``trust`` Trust level used to certify results.

- ``apprestrict`` Matching restrictions. App or group of apps that can be matched. Leave null value to disable check.

- ``datasetrestrict`` Matching restrictions. Dataset or group of datasets that can be matched. Leave null value to disable check.

- ``requesterrestrict`` Matching restrictions. Requester or group of requesters that can be matched. Leave null value to disable check.

- ``salt`` A random value to ensure order uniqueness.

- ``sign`` cryptographic signature of the order. Must be performed by the scheduler specified in the workerpool smartcontract for the order to be valid.

**RequesterOrder**

.. literalinclude:: sources/IexecODBLibOrders.sol
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

- ``callback`` Address to callback with the results (following EIP1154 interface). Leave empty (null) if no callback is required. Learn more about the `callback mechanism`_.

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
-------------------

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
-----------------------------

**[Requester] How do I enable PoCo's certification of results?**

A requester can enable the trust layer of the PoCo by setting the trust value in que requestorder. As described `here <poco-trust.html>`__, the trust can be customized depending on the required confidence level. If the requester wants à 99.99% confidence level on the results, it must set the ``trust`` field to ``10000``

**[Requester] How do I run a non determinist application despite the PoCo requiering it?**

The PoCo requires an application to be deterministic for the replication layer to provide trust in the result. If an application is not deterministic, consensus cannot be achieved and replication must be avoided.

In order to obtain a result, the requester must prevent replication by asking a ``trust`` value of ``0``. If the requester want to get some result protection, and if the application is compatible with it, the requester can the task to run in an enclave by setting the ``tag`` to ``0x1``.

**[Requester] How do I protect my result using encryption?**

The result of an execution can be valuable to the end user, and the requester might want to protect this result from leaking by encrypting it.

TODO

.. In order to do so the beneficiary must first generate an encryption key (Kb) that will be used to secure the data. This key is then send (with the identifyer of the application authorized to retreive it) to a Secret Management Service (SMS) that runs in a protected enclave. The beneficiary should also register the address of the SMS where (Kb) is available in the onchain SMSDirectory.

.. Once this setup process is done, the requester just needs to ask for the execution to be done in an enclave (by setting the ``tag`` to ``0x1``) and specify the key to use by setting the ``beneficiary`` field.

**[Dataset owner] How do I limit the usage of my dataset to a specific application**

iExec's datawallet and datastore provide all the features to monetize a valuable dataset while protecting it. Before uploading a dataset you should encrypt it using the iExec SDK. Through this process, the encryption key becomes the valuable data that has to be protected.

For applications to use your dataset you must grant them permission to retreive the encryption key. This is done when you upload the encryption key to a Secret Management Service (SMS). This prevents a malicious application from getting the encryption key and leaking the sensitive data.

You should also


**[Scheduler] Whitelist deterministic applications**

**[Scheduler] Propose a safe context for unknown applications**
