Introduction
============

What is iExec?
--------------

iExec is a multi-sided platform for computing resources.
It is a network connecting providers and users of distributed computing power, data-sets and developers,
encouraging an ecosystem of decentralized and autonomous, privacy-preserving applications.

iExec strengthens applications running on Ethereum smart contracts,
allowing for off-chain computation services and data sets on-demand.
This is made possible by the iExec Proof-of-Contribution or ‘PoCo’ consensus protocol that validates off-chain computations.


For resources providers
-----------------------

Resource providers are organized in a similar way to miners in a cryptocurrency mining pool.
One master machine, called a ‘Scheduler’, manages the allocation of computational tasks
and rewards for all connected ‘Workers’ in a ‘Worker Pool’.


What is an iExec Worker?
~~~~~~~~~~~~~~~~~~~~~~~~

| As a Worker, you can connect your machine to the network and contribute computing power, executing computational tasks in exchange for RLC tokens.
| Workers can be individuals or companies. Anyone who owns computing resources can make them available by joining a ‘Worker Pool’ and contributing to execute computational tasks in exchange for RLC.
| Any machine, from individual laptops to large-scale servers, can join a Workerpool.
| Like blockchain miners, they want a simple solution that will make their computer part of a large infrastructure
that will take care of the everything else for them.

Go to the `Be a worker`_ section to learn how to deploy your worker software.

.. _Be a worker: /worker.html

What is an iExec Workerpool?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Worker Pools are made up of multiple individual machines.
| Worker Pools are led by a ‘Scheduler’, one machine that organizes the workload and lists available WorkOrders on the iExec Marketplace to be purchased.
| Workerpools can either be public for anyone to join, or private.
| Schedulers, while not doing the actual computation, receive a fee for the management of the Worker Pool.
| Schedulers compete to attract Workers to their Worker Pool by providing efficient management and guaranteeing the earnings of Workers.

| Go to the `Be a worker pool`_ section to learn how to deploy your worker pool software.

.. _Be a worker pool: /workerpool.html

Check out the worker pool present on the market at https://pools.iex.ec


For Dapp users
--------------

| As a Dapp user, you can browse the iExec Dapp Store; a listing of user-submitted decentralized applications that are already powered by iExec.
| iExec is required as blockchain offers very limited computing capacities to run decentralized applications: (few kilobytes of storage, very inefficient virtual machines and very high latency protocol.
| As the demand for decentralized applications grows, there is an ever-growing need to provide additional computing capacity to run them.
| The existing clouds cannot fulfill the requirements for DApps that need fully decentralized infrastructures for their execution. For these applications, ‘off-chain’ computation is needed.
| iExec provides the infrastructure for this as well as ‘PoCo’ (proof-of-contribution), the consensus protocol that provides blockchain-level certification and validation of the off-chain computation.

Pick a Dapp on our Dapp store (https://dapps.iex.ec) and learn in the `Submit a Work Order`_ section how to run dapps with the iExec SDK.

.. _Submit a Work Order: /ordersubmit.html

For developers
--------------

| Developers can monetize applications by setting a fixed-fee for access using the iExec Pay-per-task model
| As a developer of decentralized applications, you can use iExec to overcome blockchain limitations of cost
 and performance by executing your computations “off-chain” on the iExec distributed infrastructure.
| Developers benefit from being able to easily adjust resource allocation, while not having to maintain any servers,
 developers still have the opportunity to rapidly upscale/downscale based on user-demand.

Go to the `DEVELOP`_ section to learn how to build Dapps (binaries or docker images) for the community

.. _DEVELOP: /dockerapp.html


For Data providers
------------------

| *Coming May 2019*
| Data providers own valuable data-sets that have been made available for use by iExec Dapps.
| Technologies such as Intel SGX and IBM Datashield combined with the pay-per-task model,
 the data wallet presents new opportunities to create highly secure applications, respecting privacy and ownership.


For requestors
--------------

| In the iExec Marketplace, LINK? (https://market.iex.ec) with just a few clicks, you can access a large amounts
 of computational power. and ready-to-use applications. and the dapp store (https://dapps.iex.ec)
| In the iExec Dapp Store, you can access an extensive list of ready-to-use decentralized applications.
| The multi-sided market provided by iExec allows to trade applications, the requestors set requestorder
 for an “ask” publication, a computing resources provider should accept the deal and process this requestorder
  at the price fixed by the requestor.
| The Pay-Per-Task model and the blockchain allow a high level of control of operating costs and expenses for computation.
| Requestors can define a minimum level of trust to ensure the work they receive has been correctly processed,
 whether or not to execute on Intel SGX hardware enclave enables machines, for example.
| Requestors select resources providers depending on certain criteria such as geographical location or energy management.
| Requestors can monitor and fully audit all computing activity, thanks to Blockchain.