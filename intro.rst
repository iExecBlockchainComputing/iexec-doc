Introduction
============

| Comparably to the oil market, the iExec marketplace offers a uniform and standardized access to computing resources, regardless of their provider.
| iExec strengthens applications running on Ethereum smart contracts, allowing for off-chain computation services and datasets on-demand.
| This is made possible by the iExec Proof-of-Contribution or PoCo consensus protocol that validates off-chain computations.


For resources providers
-----------------------

| Resource providers are organized in a similar way to miners in a cryptocurrency mining pool.

iExec Workerpool
~~~~~~~~~~~~~~~~

| **Workers** are organized in **worker pools**, where a lead computer schedules and manages the work distribution.
| Workers are the machines in charge of executing the tasks requested  by users, **the requester**.
| Similarly to blockchain miners, workers share their resources and get rewarded in RLC, the iExec's cryptocurrency and means of payment.

| There are two types of **worker pools**:
|    - Public worker pools
|    - Private worker pools

| Public worker pools are open for anyone to join.
| Anyone can also deploy a worker pool for others to join.
| In the marketplace, you will also find private workers pools: for instance, a cloud provider running their own scheduler and making available their own machines.
| iExec has sealed agreement deals with several cloud companies, be it household names like IBM or TFCloud, or smaller providers in the area of Green IT.

| In this open and inclusive system, competitive forces drive worker pools to offer the best quality of service possible,
 and a dynamic reputation mechanism assigns a score to each worker completing a work.


| Go to the `Be a worker pool`_ section to learn how to deploy your worker pool.

.. _Be a worker pool: /workerpool.html

Browse the available Worker Pools: https://pools.iex.ec/


iExec Worker
~~~~~~~~~~~~

| Anyone who owns computing resources can make them available by joining a **Worker Pool** and contributing to execute computational tasks in exchange for RLC.
| Any machine, from individual laptops to large-scale servers, can join a **Worker pool**.
| Like blockchain miners, they want a simple solution that will make their computer part of a large infrastructure that will take care of the everything else for them.
| As a worker, if you switch to a different worker pool, you will still be able to maintain your reputation score, bringing it "with you" to your new pool, as this is all recorded on the blockchain.


Go to the `Be a worker`_ section to learn how to deploy your worker software.

.. _Be a worker: /worker.html


For Dapp users
--------------

| As a Dapp user, you can browse the iExec Dapp Store; a listing of user-submitted decentralized applications that are already powered by iExec.
| iExec is required to enhance the limited computing capacities of blockchain to run compute-intensive decentralized applications: few kilobytes of storage, very inefficient virtual machines and very high latency protocol.
| As the demand for decentralized applications grows, there is an ever-growing need to provide additional computing capacity to run them.
| The existing clouds cannot fulfill the requirements for DApps that need fully decentralized infrastructures for their execution.
| For these applications, ‘off-chain’ computation is needed.
| iExec provides the infrastructure for this as well as ‘PoCo’ (proof-of-contribution), the consensus protocol that provides blockchain-level consolidation and validation of the off-chain computation.

| Pick a Dapp on our Dapp store (https://dapps.iex.ec)
| and learn in the `Task execution`_ section how to run dapps with the iExec SDK.

.. _Task execution: /ordersubmit.html

| In the iExec Marketplace https://market.iex.ec, with few clicks, you can access a large amounts of computational power.
| In the iExec Dapp Store https://dapps.iex.ec, you can access an extensive list of ready-to-use decentralized applications.

For developers
--------------

| Developers can monetize applications by setting a fixed-fee for access using the iExec Pay-per-task model
| As a developer of decentralized applications, you can use iExec to overcome blockchain limitations of cost
 and performance by executing your computations “off-chain” on the iExec distributed infrastructure.
| Developers benefit from being able to easily adjust resource allocation, while not having to maintain any servers,
 developers still have the opportunity to rapidly upscale/downscale based on user-demand.

Go to the `Provide Application`_ section to learn how to build Dapps (binaries or docker images) for the community

.. _Provide Application: /dockerapp.html


For Data providers
------------------

| *Coming May 2019*

| Data providers own valuable datasets that have been made available for use by iExec Dapps.
| Technologies such as Intel SGX and IBM Datashield combined with the pay-per-task model,
 the data wallet presents new opportunities to create highly secure applications, respecting privacy and ownership.


For requesters
--------------

| The multi-sided market provided by iExec allows to trade applications, the requesters set requestorder for an “ask” publication, a computing resources provider should accept the deal and process this requestorder
  at the price fixed by the requester.
| The Pay-Per-Task model and the blockchain allow a high level of control of operating costs and expenses for computation.
| Requesters can define a minimum level of trust to ensure the task has been correctly processed, whether or not to execute on Intel SGX hardware enclave enables machines, for example.
| Requesters select resources providers depending on certain criteria such as geographical location or energy management.
| Requesters can monitor and fully audit all computing activity, thanks to Blockchain core property.
