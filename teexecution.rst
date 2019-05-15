End-To-End Encryption
==================================
iExec End-To-End Encryption with Secret Management Services (SMS)

Input dataset encryption
----------------------------------

As a dataset provider, you might want to protect your dataset with encryption in order to monetize it.
Any encrypted dataset will be decrypted on worker resources with a dataset secret key retrieved from the Secret Management Service.
This dataset secret key need to be created and push by the dataset owner.
At this point, the decrypted dataset will be ready to be used by the app.

# How to encrypt a dataset

TODO

Result encryption
----------------------------------

You can choose to get an encrypted result depending on the privacy of your task result.

* If the beneficiary of the task is set, the result will be pushed to the iExec Result Repository:
  - If the key of the beneficiary has been pushed to the Secret Management Service, the result will be encrypted before being pushed to the Result Repository. The beneficiary of the task will be the only one able to access and decrypt the result.
  - If the key of the beneficiary is missing in the Secret Management Service, the result will be pushed to the Result Repository without encryption. The beneficiary of the task will be the only one able to access the result.
* If the beneficiary of the task is unset, the result will be pushed to IPFS without encryption.


# How run a task with an encrypted result

1. Generate your beneficiary keys

.. code:: bash
        
    iexec tee generate-beneficiary-keys

2. Push your keys to the SMS 

Please check your 'chain.json' file contains an entry '"sms": "https://kovan-pool.iex.ec:443"'

.. code:: bash
        
    iexec tee push-secret

3. Buy computation with your beneficiary address

market.iex.ec: Advanced parameters > Beneficiary > Private
SDK: iexec.json > requestorder > beneficiary > 0xyourAddress

4. Download the result and decrypt it

.. code:: bash
        
    iexec task show <0xtask> --download
    iexec tee decrypt-results <encryptedResultsPath>



.. include:: contactus.rst
