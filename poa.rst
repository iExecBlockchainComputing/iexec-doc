Consortium Deployment
=====================


Introduction
------------



The iExec Enterprise Edition (iEE) addresses the needs of enterprises and consortiums that want to improve their processes and create businesses by leveraging blockchain paradigms.

iEE refers to private, consortium, and hybrid implementations of the iExec stack for business applications.
This integration and consulting service allows corporates to optimize their computing infrastructure, orchestrate data sharing and deploy applications with no downtime, fraud, or third-party tampering.

This will allow companies to **automate their businesses through a transparent history, immutable, and cryptographically verifiable state.**
Automate their business between them but also allow them to target new customers thanks to their joint forces and efficient services on their consortium chain.

You can see further explanation and illustration of this consortium usage presented in the `iExec talk at ETHCC <https://github.com/iExecBlockchainComputing/iexec-deploy/blob/master/poa/slides/README.md/>`_


We define a consortium as a set of privates keys that will be used to run their "home" chain.
Those keys will create what we call Trusted Authority that will be in charge of creating blocks in their home chain.
The public consensus (Proof of work, Proof of stake) is replaced with a Proof of Authorities consensus. The consensus used in our case is `Aura <https://wiki.parity.io/Aura>`_ .
If all keys are held by only one entity, this is a basically a private chain, because there is only one operator. This operator is able to rewrite or erase all history.
Nevertheless, this operator can choose to give access to his blockchain state to others and allow them eventually to send transactions to it too.
A better approach is to share those authorities' private keys between different legals entities (2, 3, n companies). This is what we call a consortium chain.
This is the responsibility of the consortium to establish an initial ceremony to launch the genesis of the chain.
As a public user, you still have to trust that this consortium does not collude to rewrite or erase the chain if you used it. But as operators are more than one and have different incentives,
this is a way to mitigate governance centralization.

This is ultimately a Performance/Security/cost trade-off: you still have the choice to use a public chain and pay the gas price, or use mitigated governance but efficient services proposed by consortium chains.


In their "home" chain, consortiums will deploy the iExec stack of smart contracts, and will be able to use it in their isolated context and without public gas transaction cost or fixed gas.
They will also be able to add custom permission access rights according to their consortium policy.
Several permissions levels can be setup:
- Who can read the state of the blockchain,
- Who can interact with existing smart contract on the blockchain,
- Who can create new smart contract on chain.
See more details on permissioning setup `here <https://wiki.parity.io/Permissioning/>`_  .

iExec off-the-shelf smart contracts provide on-chain live and auditable resource registries (dataset, application, servers) inside the consortium.
In addition, you gain an off-chain auditability feature between parties thanks to the PoCo (https://docs.iex.ec/poco.html) and iExec Middleware: Schedulers and Workers.
That means that all logged transactions can then be used between parties to establish audit reports or even payment according to their respective usage.

This is the first step towards automation. The next step can be to also add automatic payment thanks to the RLC token. This is where the introduction of bridges can enable to transfer an asset into the consortium chain and then use it as a payment commodity.
Consortium parties will deposit and withdraw between mainnet and their home consortium chain, the amount they wanted to use.
We can see the mainnet as a cold chain and their consortium chain as hot chain. Hot Chain with no public security (like Proof Of Work) but providing more efficient and fast services.
Taking the analogy of cold wallet, hot or burner wallets but here for "cold chain" ,hot chain...

To achieve the PoA chain feature, we can use the Ethereum Parity client parity in PoA setup mode.
The deployment `parity-deploy <https://github.com/paritytech/parity-deploy/>`_  tools can help to easily setup consortium nodes, keys and configs.
Both pieces of software, `parity-deploy` and `parity-ethereum` client are provided by `Parity Technologies <https://github.com/paritytech/>`_

To achieve bridge features, we used softwares provided by `PoA Network <https://github.com/poanetwork/>`_
as it provides an ERC-20 to ERC-20 token bridge.
We will detail such a configuration in the following paragraphs.


Hand zone
----------

Prerequisite
~~~~~~~~~~~~
- use ubuntu OS or ubuntu docker image:

Prerequisite lib packages

.. code-block:: bash

    docker run -it --name poa_config ubuntu:18.10 /bin/bash
    apt-get update
    apt-get install -y zip sudo git curl wget gcc


Install some eth key libs

.. code-block:: bash

    git clone https://github.com/paritytech/parity-ethereum -b v2.4.5
    cd parity-ethereum/
    curl https://sh.rustup.rs -sSf | sh -s -- -y
    source $HOME/.cargo/env
    cargo build -p ethkey-cli --release
    cargo build -p ethstore-cli --release
    ./target/release/ethkey --help
    ./target/release/ethstore --help
    cp -f ./target/release/ethkey /usr/bin/
    cp -f ./target/release/ethstore /usr/bin/
    cd -


PoA Chains
~~~~~~~~~~

We use the `parity-deploy` tool, to facilitate the configuration and key generation for the PoA chain.

.. code-block:: bash

    git clone https://github.com/iExecBlockchainComputing/parity-deploy.git -b v1.0.4

You will need to define the number of nodes for your Authority, the password length (`openssl rand -base64` is used) and the version of the Parity Ethereum client you want to use.

.. code-block:: bash

    export NB_NODES=5
    export PASSWORD_LENGTH=12
    export PARITY_DOCKER_VERSION=v2.4.5
    export USER=$(whoami)

You can first generate your authority's keys with the command:

.. code-block:: bash

    cd parity-deploy
    ./config/utils/pwdgen.sh -n ${NB_NODES} -l ${PASSWORD_LENGTH}


It generates a password file in deployement/[1->5]/password. It will be used to encrypt the authority nodes' wallet files when launching subsequent commands.

Then you can generate the number of nodes you want to bootstrap with the command:

.. code-block:: bash

    cp /usr/bin/ethkey .
    cp /usr/bin/ethstore .
    ./parity-deploy.sh --config aura --name HOME-CHAIN --nodes ${NB_NODES} --entrypoint "/bin/parity" --release $PARITY_DOCKER_VERSION --expose


You can then customize the `deployment/chain/spec.json` generated.
- add account with funds
- define `stepDuration` for `blocktime`
- define the `networkID` of your chain

You can now zip the config:

.. code-block:: bash

    zip -r poa-config.zip .env docker-compose.yml deployment data config


If you used a ubuntu docker container, copy the zip to your host. From your host run (you need docker):

.. code-block:: bash

    export ID_CONTAINER=$(docker ps --format '{{.ID}}' --filter "name=poa_config")
    docker cp ${ID_CONTAINER}:/parity-deploy/poa-config.zip .


Test your generated configs on your host (you need docker and docker-compose):

.. code-block:: bash

    unzip poa-config.zip
    docker-compose up

You must see all your nodes starting with no errors and each node connected to 4 other peers.

.. NOTE::

    In this generated config, you may need to edit manually some files according to your network in `docker-compose.yml`:

        - changing ports to avoid conflicts

        - adding command start option like `--force-sealing --logging levels`



PoA Bridges
~~~~~~~~~~~

Poa-Network bridges allow to bridge ERC-20 to ERC-20 token as described in the poa-network medium article :

https://medium.com/poa-network/introducing-the-erc20-to-erc20-tokenbridge-ce266cc1a2d0

The 3 main bridge components are smart contracts, bridge agent software, and interface to use by users to the bridged asset.
There are 2 parts for the smart contract. A bridge smart contract deployed on the mainnet and another bridge smart contract deployed on the home chain.


Bridge contracts:

https://github.com/poanetwork/poa-bridge-contracts.git

Bridge Agents:

https://github.com/poanetwork/token-bridge.git

Bridge Interface:

https://github.com/poanetwork/bridge-ui

You can see a full deployment script for the poa-network bridge stack on PoA chains iExec context added with RLC token and PoCo smart contracts deployed.

https://github.com/iExecBlockchainComputing/iexec-deploy/blob/master/poa/CI/bootpoatestnetV3master.sh

We are planning to merge soon a new feature that will allow adding a whitelisting feature as described here.

https://forum.poa.network/t/consortium-bridge/1739


Contact us to have more information about this.

     - mail support@iex.ec
     - slack iexec-team.slack.com
