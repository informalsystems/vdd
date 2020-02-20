*** This is the beginning of an unfinished draft. Don't continue reading! ***

# Tendermint Blockchain

> Rough outline of what the component is doing and why. 2-3 paragraphs

A blockchain is a growing sequence (called log) of sets of transactions. 
If several processes in a distributed system have access to the blockchain, they can (1) provide transactions as input and (2) use the log to execute these transactions in order. 
This then allows to implement state machine replication.

The log itself should be implemented in a reliable way, which introduces the need for fault-tolerance and distribution.
Tendermint implements the blockchain over an unreliable (Byzantine) distributed system. 
More precisely, a Tendermint system consists of many full nodes that each maintain a local copy of the (prefix of the current) log.

In this specification, we are only concerned with the logs. 
They are maintained at so-called _full nodes_, and are the result of the execution of the Tendermint consensus protocol (described in the ArXiv paper). 
The Tendermint consensus implements a loop with iterator _h_ (height). 
In each iteration, upon deciding on the transactions to be put into the log, a new log entry is created.


# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc.

> should give the reader the understanding in what environment this component
will be used.

This specification is central in the collection of Tendermint protocols. The behavior of protocols like fastsync (**link**), or the lite client (**link**) will be defined with respect to this specification. E.g., the lite client implements a read operation of the log entry of some height _h_. It is thus crucial to understand what data is stored in this log entry, and what are the precise semantics of a read operation in a faulty environment.


## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view.

A blockchain provides a growing sequence of sets of transactions.

## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

**[TMBC-SEQ]** The Tendermint blockchain is a list of headers with heights:

 - A header is a record of data (the structure of this data will be refined below)

 - Heights are strictly increasing.

 - If the blockchain contains a header of height *h*, then for all _h'<h_
it contains a header of height _h'_.

 - During operation, new headers may be appended to the list one by one.

A header is a record, containing the following fields
# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly

> should have clear formalization in temporal logic.

## Timing, nodes, and correctness assumptions

In Tendermint, the agents who choose to interact with each other collectively implement the log in a distributed manner by executing a protocol. 
At the same time, the agents may benefit and harm others by deviating from the expected behavior. 
As a result, the agents should be motivated (incentivized) to follow the protocol. 
This is done by proof of stake. 
In order to participate, the agents bond atoms. 
If they misbehave, the agents are punished. 
Before the agents can claim their atoms, they have to wait for a certain period of time, called the _unbonding period_. 
The unbonding period is a configuration parameter. 
Moreover, the influence an agent has depends on the amount of bonded atoms, which is called its _voting power_.

**[TMBC-TIME_PARAMS]** A Tendermint blockchain has the following configuration parameters:
 - _unbondingPeriod_: a time duration.
 - _trustingPeriod_: a time duration less than _unbondingPeriod_.

**[TMBC-NODES]** Tendermint full nodes (or just "full nodes"), execute a set of protocols, e.g., consensus, gossip, fast sync, etc. 
The full nodes that actively participate in the distributed consensus are called _validator nodes_.

**[TMBC-CORRECT]**
We define a predicate _correct(n, t)_, where _n_ is a node and _t_ is a 
time point. 
The predicate _correct(n, t)_ is true if and only if _n_ follows all the protocols until time _t_.


## Blockchain data structure

**[TMBC-VALIDATOR]** Given a full node, a _validator_ is a pair _(public key, voting power)_, where 
  * _public key_ is the public key of the full node, 
  * _voting power_ is an integer (representing the full node's voting power in a certain consensus instance).

Remark: We observed that the term _validator_ refers to both a data structure and a full node that participates in the distributed computation. Therefore, we introduce the notions _validator pair_ and _validator node_, respectively.

**[TMBC-BLOCK]** Each block contains (among others) the following fields
 -  _Validators_: a set of validator pairs (as defined in **[TMBC-VALIDATOR]**), of the validator nodes that participated in the consensus instance which generated the block,
 - _NextValidators_: a set of validator pairs of the full nodes that participate in the consensus instance of the next block.
 - _bfttime_: the real-time at which the block is generated

**[TMBC-TIME]** _bfttime_ corresponds to the reading of the local clock of a validator (how this time is computed may change when the Tendermint consensus is modified). In this note, we assume that all clocks are synchronized to realtime (and so is bfttime). We can make this more precise eventually (incorporating clock drift, accuracy, precision, etc.). Right now, we consider this assumption sufficient, as clock synchronization (under NTP) is in the order of milliseconds and _trustingPeriod_ is in the order of weeks.

## Failure model

**[TMBC-FM-CONS]** 
There is a subset _C_ of validators in _NextValidators_ at height _h_ that holds more than 2/3 of the total voting power in _NextValidators_ at height _h_  such that every validator in _C_ follows the consensus protocol until consensus for height _h+1_ is terminated. (Consensus failure model)


**[TMBC-FM-2THIRDS]** If a block _h_ is generated at time _bfttime_, then there is a subset _C_ of validators in _NextValidators_ at height _h_ that holds more than 2/3 of the total voting power in _NextValidators_ at height _h_  such that every validator in _C_ follows *all* protocols until _bfttime + trustingPeriod_. (Tendermint Failure Model)

Formally,
\[
\sum*{(v,p) \in h.Header.NextV \wedge correct(v,h.Header.bfttime + trustingPeriod)} p >
2/3 \sum*{(v,p) \in h.Header.NextV} p
\]

_Remark:_ The definition of correct [TMBC-Corr] refers to realtime, while it is used here with bfttime and trustingPeriod, which are "hardware times". Due to [TMBC-TIME], we do not make a distinction here. 




## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

**[TMBC-D-VAL]** There is a predicate _Valid()_ defined over blocks.
(perhaps we do not need that now)

### Temporal Properties

> safety specifications / invariants in English

> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually.

Each correct full node maintains a log. 
A log is a sequence of blocks.

A block is a data structure described in
https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md



**[TMBC-VC_AGR]** It is always the case that, for any two correct full nodes _i_ and _j_, either _i.log_ is a prefix of _j.log_ or _j.log_ is a prefix of _i.log_.

**[TMBC-VC-PROG]** For all correct validators _i_ and all times _t_ there exists a time _t'_, such that  _|log_i(t)| < |log_i(t')|_.

**[TMBC-VC_VAL]** Not clear. 
Application specific. 
We will need to fix that eventually. 
I guess for fastsync and liteclient we don't need it.
(It is always the case that each log entry _b_ satisfies _Valid(b)_. not
  sure. _Valid_ may take the current state)
(_Remark:_ Validity should make reference to the mempool, e.g., only messages from the
  mempool + we will need a spec for the mempool. For now I leave it like that
  as the liteclient and fastsync do not care about that.).



> How is the problem statement linked to the "Sequential Problem statement".
Simulation, implementation, etc. relations

## Definitions

> In this section we become more concrete, with basic (abstracted) data types

> some math that allows to write specifications and pseudo code solution below.
Some variables, etc.

For the remainder we refer to arXiv paper.

*decision_p[]* ... implements the log at full node _p_.

## Solution

> Basic data structures. Simplified, so that we can focus on the distributed
algorithm here. If existing: link to Tendermint data structures, and mentioned
if details were omitted.



### Outline

> Describe solution (in English), decomposition into functions, where communication to other components happens.

### Details

> Pseudo code of the solution


## Correctness arguments

> Proof sketches of why we believe the solution satisfies the problem statement.
Possibly giving inductive invariants that can be used to prove the specifications
of the problem statement
