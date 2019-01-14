Replication & Trust
-------------------

How to achieve trust ?
~~~~~~~~~~~~~~~~~~~~~~

| The PoCo offers a consensus mechanism that can certify the likelihood of a result to be valid.
| This consensus relies on the scoring of workers and the replication of a task's execution to combine the score of the workers that come up with the same result.
| This consensus is largely based on Sarmenta's work [Sarmenta2002]_ with specific tuning of the scoring function [Trust2018]_.


Contribution credibility
~~~~~~~~~~~~~~~~~~~~~~~~

| Each worker's contribution has an associated credibility. This credibility derives from the worker's history score.
| As described in [Trust2018]_, a worker score is a positive integer that is incremented for each valid and verified contribution.
| In case of bad contribution, a worker loses one third of it's score.

This credibility can be expressed as a likelihood percentage but also as a weight value that can be used to detect consensus without resorting to floating point arithmetics,
 more details in [Trust2018]_.

Requiring a trust level
~~~~~~~~~~~~~~~~~~~~~~~

Based on [Sarmenta2002]_ describing the way to combine worker's contribution and to evaluate a result's likelihood,
 a requester can ask for the level of trust as a input for the PoCo processing, to impose a certain quality of service.
The trust level corresponds to a minimum correctness likelihood that a result must achieve to be valid.
For example, a trust level of 0 means any contribution would be accepted, regardless of the score of the worker proposing it.
On the other hand a trust level of 99.99% means a result will only be accepted if the contribution towards it result shows a correctness probability higher than 99.99%.

The trust level is expressed, on-chain, by a integer ``trust`` such that ``threshold = 1 - 1 / trust``.

========= ======================================
**Trust** **PoCo enforced confidence threshold**
--------- --------------------------------------
0         0%
1         0%
2         50%
100       99%
10000     99.99%
1000000   99.9999%
========= ======================================

Limitations
~~~~~~~~~~~

A limitation of this consensus mechanism is that it requires the application to be replicable.
Non deterministic applications, of long running jobs such as webservers, do not meet this requirement out of the box.
When it is possible, the application developer must provide a deterministic result in the ``/iexec/consensus.iexec`` file.
Otherwise, the user can still run its application on the iExec platform but would have to disable the PoCo's consensus layer and rather rely on hardware security.

References
~~~~~~~~~~

.. [Sarmenta2002] Luis F.G.Sarmenta. Sabotage-tolerance mechanisms for volunteer computing systems. 2002. Future Generation Computer Systems, 18(4), 561–572 `PDF <http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.2962&rep=rep1&type=pdf>`_

.. [Trust2018] Trust management in the Proof of Contribution protocol. 2018. Technical report. `PDF <https://github.com/iExecBlockchainComputing/iexec-doc/raw/master/techreport/iExec_PoCo_and_trustmanagement_v1.pdf>`_
