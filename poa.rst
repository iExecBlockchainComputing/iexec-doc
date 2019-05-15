Consortium Deployment
=====================


Introduction
------------



The iExec Enterprise Edition (iEE) addresses the needs of enterprises and consortium that want to improve their processes and create businesses by leveraging blockchain paradigms.

iEE refers to private, consortium, and hybrid implementations of the iExec stack for business applications.
This integration and consulting service allow corporates to optimize their computing infrastructure, orchestrate the sharing of data and deploy applications with no downtime, fraud, or third-party interference.

This will allow companies to **automate their businesses through a transparent history, immutable, and cryptographically verifiable state.**
Automate their business between them but also allow them to target new customers thanks to their join forces and efficient services on their consortium chain.

You can see explanation and illustration this consortium usage presented in the `iExec talk at ETHCC <https://github.com/iExecBlockchainComputing/iexec-deploy/blob/master/poa/slides/README.md/>`_


We define a consortium as a set of privates keys that will be used to run their "home" chain.
Those keys will create what we call Trusted Authority that will be in charge of creating blocks in their home chain.
The public consensus (Proof of work, Proof of stake) is replaced with a Proof of Authorities consensus. The consensus used in our case is `Aura <https://wiki.parity.io/Aura>`_ .
If all keys are hold by only one entity. This is a basically private chain. Because there is only one operator. This operator is able to rewrite or erase all history.
Nevertheless, this operator can choose to give access to his blockchain state to others and allow them eventually to send transactions to it too.
A better approach is to share those authorities privates keys between different legals entities (2,3,n companies). This is what we call a consortium chain.
This is the responsibility of the consortium to establish an initial ceremony to launch the genesis of the chain.
As a public user, you still have to trust that this consortium do not collude to rewrite or erase the chain if you used it. But as operators are more than one and have different incentives,
this is a way to mitigate governance centralization.

This is ultimately a Performance/Security/cost trade-off to choose, you still have the choice to use public chain and pay the gas price or used mitigate governance but efficient service proposed by consortium chains.


In their "home" chain, consortium will deploy iExec stack smart contract and will be able to used it in their restraint context and without public gas transactions cost or fixed gas.
They will also be able to add custom permission access rights according to their consortium policy.
Several permissions levels can be setup :
- Who can read the state of the blockchain,
- Who can interact with existing smart contract on the blockchain
- Who can create new smart contract on the chain
See more details on permissioning setup `here <https://wiki.parity.io/Permissioning/>`_  .

iExec off-the-shelf smart contracts provide on-chain live and auditable resources registries (dataset, application, servers) inside the consortium.
In addition, you gain an off-chain auditability usages between parties thanks to the PoCo (https://docs.iex.ec/poco.html) and iExec Middleware: Schedulers and Workers.
That means that all logged transactions can then be used then between parties to establish audit report or even payment according to respective usages.

This is the first step of automation. The next step can be to also add automatic payment thanks to RLC token. This is where the introduction of bridges can enable
to transfer asset into the consortium chain and then used it as a payment commodity.
Consortium parties will deposit and withdraw between mainnet and their home consortium chain, the amount they wanted to use.
We can see the mainnet as a cold chain and their consortium chain as hot chain. Hot Chain with no public security (like Proof Of Work) but more efficient and fast services.
Taking the analogy of cold wallet, hot or burner wallets but here for "cold chain" ,hot chain ...

To achieve the PoA chain feature, Ethereum Parity client parity in PoA setup mode can be used.
The deployment `parity-deploy <https://github.com/paritytech/parity-deploy/>`_  tools can help to easily setup consortium nodes keys and configs.
Both softwares, parity-deploy and parity-ethereum client are provide by `Parity Technologies <https://github.com/paritytech/>`_

To achieve bridges features, we used softwares provides by `PoA Network <https://github.com/poanetwork/>`_
as it provide ERC-20 ERC-20 token bridge.
We will detail how to configure this more in details in the followings paragraphs.


Hand zone
----------

Prerequisite
~~~~~~~~~~~~
- use ubuntu OS or ubuntu docker image :

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

We use parity-deploy tooling, to facilitate the config and keys generation of PoA chain.

.. code-block:: bash

    git clone https://github.com/iExecBlockchainComputing/parity-deploy.git -b v1.0.4

You define the number of nodes for your Authority, the password length (openssl rand -base64 is used) and the version of Parity Ethereum client you want to used.

.. code-block:: bash

    export NB_NODES=5
    export PASSWORD_LENGTH=12
    export PARITY_DOCKER_VERSION=v2.4.5
    export USER=$(whoami)

You can first generate your authorities key with the command :

.. code-block:: bash

    cd parity-deploy
    ./config/utils/pwdgen.sh -n ${NB_NODES} -l ${PASSWORD_LENGTH}


It generate a password file in deployement/[1->5]/password. It will be used to encrypt wallet files of nodes authorities when launching next commands.

Then you can generate the numbers of nodes you wanted to bootstrap with the commande  :

.. code-block:: bash

    cp /usr/bin/ethkey .
    cp /usr/bin/ethstore .
    ./parity-deploy.sh --config aura --name HOME-CHAIN --nodes ${NB_NODES} --entrypoint "/bin/parity" --release $PARITY_DOCKER_VERSION --expose


You can customize then the deployment/chain/spec.json generated.
- add account with fonds
- define stepDuration for blocktime
- define your networkID of your chain

You can now zip the config :
Zip your config :

.. code-block:: bash

    zip -r poa-config.zip .env docker-compose.yml deployment data config


If you used ubuntu docker container, copy the zip on your host. On your host (you need docker):

.. code-block:: bash

    export ID_CONTAINER=$(docker ps --format '{{.ID}}' --filter "name=poa_config")
    docker cp ${ID_CONTAINER}:/parity-deploy/poa-config.zip .


Test your generated configs on your host (you need docker and docker compose) :

.. code-block:: bash

    unzip poa-config.zip
    docker-compose up

You must see all your node starting with no errors and each nodes connected to 4 others peers.

.. NOTE::

    In this generated config, you may need to edit manually some files according to your network in docker-compose.yml:

        - changing ports to avoid conflicts

        - adding command start option like --force-sealing --logging levels



PoA Bridges
~~~~~~~~~~~

Poa-Network bridges allow to bridge ERC-20 to ERC-20 token as describe in the poa-network medium article :

https://medium.com/poa-network/introducing-the-erc20-to-erc20-tokenbridge-ce266cc1a2d0

The 3 mains bridges components are smart contracts, bridges agents softwares, and interface to use by users to bridge asset.
There is 2 parts for smart contract. A bridge smart contract deploy on the mainnet and another bridge smart contract deploy on the home chain.


Bridges contracts :

https://github.com/poanetwork/poa-bridge-contracts.git

Bridges Agents:

https://github.com/poanetwork/token-bridge.git

Bridge Interface :

https://github.com/poanetwork/bridge-ui

You can see a full deployment script of poa-network bridges stack on PoA chains iExec context added with RLC token and PoCo smart contracts deployed.

https://github.com/iExecBlockchainComputing/iexec-deploy/blob/master/poa/CI/bootpoatestnetV3master.sh

We planned to merge soon a new feature that will allow to add a whitelisting feature as described here.

https://forum.poa.network/t/consortium-bridge/1739


Contacts-us to have more informations about this.

     - mail support@iex.ec
     - slack iexec-team.slack.com
