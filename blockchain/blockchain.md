*** This is the beginning of an unfinished draft. Don't continue reading! ***

# Tendermint Blockchain

> Rough outline of what the component is doing and why. 2-3 paragraphs

A blockchain is a growing list of sets of transactions, denoted by _decision_. 
If several processes in a distributed system have access to the blockchain, they can (1) provide transactions as input and (2) use the decision list to execute these transactions in order. 

The _decision_ list itself should be implemented in a reliable way, which introduces the need for fault-tolerance and distribution.
The Tendermint protocols implement the blockchain over an unreliable (Byzantine) distributed system. 
More precisely, a Tendermint system consists of many full nodes that each maintain a local copy of the (prefix of the current) _decision_ list.

In this specification, we are only concerned with the _decision_ lists. 
They are maintained at so-called [full nodes][fullnode], and are the result of the execution of the Tendermint consensus protocol (described in the ArXiv paper). 
The Tendermint consensus implements a loop with iterator _h_ (height). 
In each iteration, upon deciding on the transactions to be put into the _decision_ list, a new _decision_ list entry is created.


# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc.

> should give the reader the understanding in what environment this component
will be used.

This specification is central in the collection of Tendermint protocols. 
The behavior of protocols like [fastsync][fastsync], or the [light client][lightclient] will be defined with respect to this specification. 
E.g., the light client implements a read operation of the _decision_ list entry of some height _h_. 
It is thus crucial to understand what data is stored in this _decision_ list entry, and what are the precise semantics of a read operation in a faulty environment.


## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view.

A blockchain provides a growing sequence of sets of transactions.

## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

#### **[TMBC-HEADER]**:
A set of blockchain transactions is stored in a data structure called _block_, which 
contains a field called _header_. (The data structure _block_ is defined [here][block]). 
As the header contains hashes to the relevant fields of the block, for the purpose of this
specification, we will assume that the blockchain is a list of headers, rather than a
list of blocks. 
We also assume that every hash in the header identifies the data it hashes. 
Therefore, in this specification, we do not distinguish between hashes and the 
data they represent.

#### **[TMBC-SEQ]**:
The Tendermint blockchain is a list of headers. 
 - A header is a record of data (defined [here][header]).
 - Each header has an integer called a _height_.
 - Heights are strictly increasing.
 - If the blockchain contains a header of height _h_, then for all _h'<h_
it contains a header of height _h'_.
 - During operation, new headers may be appended to the list one by one.

# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly

> should have clear formalization in temporal logic.

## Timing, nodes, and correctness assumptions

In a Tendermint system, the nodes that choose to interact with each other collectively 
implement the _decision_ list in a distributed manner by executing a protocol. 
The Tendermint protocols should ensure that the participating nodes cannot benefit by 
deviating from the expected behavior. 
As a result, the nodes should be motivated (incentivized) to follow the protocol. 
This is achieved by requiring the nodes to bond atoms and by the system punishing the nodes that do not follow the protocol. 
That is, the nodes bond atoms in order to participate, and they are punished if they misbehave. 
If the nodes want to claim their atoms, they have to wait for a certain period of time, 
called the _unbonding period_. 
The unbonding period is a configuration parameter of the system. 

#### **[TMBC-TIME_PARAMS]**:
A Tendermint blockchain has the following configuration parameters:
 - _unbondingPeriod_: a time duration.
 - _trustingPeriod_: a time duration smaller than _unbondingPeriod_.

#### **[TMBC-NODES]**:
Tendermint full nodes (or just _full nodes_), execute a set of protocols, e.g., consensus, gossip, fast sync, etc. 
When a full node actively participates in the distributed consensus, it is called a _validator node_.

#### **[TMBC-CORRECT]**:
We define a predicate _correct(n, t)_, where _n_ is a node and _t_ is a 
time point. 
The predicate _correct(n, t)_ is true if and only if the node _n_ follows all the protocols until time _t_.

### Full node invariants

#### **[TMBC-FaultyFull]**:
No assumption is made about the behavior of faulty full nodes; they may be Byzantine.

#### **[TMBC-Sign]**:
Signatures and hashes cannot be broken.


## Blockchain data structure

#### **[TMBC-VALIDATOR]**:
Given a full node, a _validator_ is a pair _(public key, voting power)_, where 
  - _public key_ is the public key of the full node, 
  - _voting power_ is an integer (representing the full node's voting power in a certain consensus instance).

Remark: We observed that the term _validator_ refers to both a data structure and a full node that participates in the distributed computation. Therefore, we introduce the notions _validator pair_ and _validator node_, respectively.

#### **[TMBC-HEADER-FIELDS]**:
TODO: is this necessary here, if we add the data structures below? \
Each header contains (among others) the following fields
 -  _Validators_: a set of validator pairs (as defined in [**[TMBC-VALIDATOR]**](TMBC-VALIDATOR-link), of the validator nodes that participated in the consensus instance which generated the block,
 - _NextValidators_: a set of validator pairs of the full nodes that participate in the consensus instance of the next block,
 - _bfttime_: the real-time at which the block is generated,
 - _LastCommit_: the set of signatures of the validators that committed the last block.

#### **[TMBC-TIME]**:
The time _bfttime_ corresponds to the reading of the local clock of a validator (how this time is computed may change when the Tendermint consensus is modified). 
In this specification, we assume that all clocks are synchronized to real-time (and so is bfttime). 
We can make this more precise eventually (incorporating clock drift, accuracy, precision, etc.). 
Right now, we consider this assumption sufficient, as clock synchronization (under NTP) is in the order of milliseconds and _trustingPeriod_ is in the order of weeks.

### Blockchain data structure invariants

#### **[TMBC-INV-SIGN]**:
The _LastCommit_ field of the block at height _h+1_ contains only signatures of 
validators. The validators that signed have more than two-thirds of the voting power at height _h_.

#### **[TMBC-INV-VALID]**:
The _NextValidator_ set of a block at height _h_ is equal to the _Validator_ set 
of the block at height _h+1_.

#### **[TMBC-INV-VALID-UNIQUE]**:
The set _Validators_ contains at most one validator pair for each full node.

#### **[TMBC-INV-NEXT-VALID-UNIQUE]**:
The set _NextValidators_ contains at most one validator pair for each full node.
 
## Failure model

#### **[TMBC-FM-CONS]**: 
(Consensus failure model)
There is a set _C_ of validator pairs, such that _C_ is a subset of _NextValidators_ at height _h_, where: 
  - The validator pairs in _C_ hold more than two-thirds of the total voting power in _NextValidators_ at height _h_
  - Every validator pair in _C_ follows the consensus protocol until consensus for height _h+1_ is terminated. 



#### **[TMBC-FM-2THIRDS]**:
(Tendermint Failure Model)
If a block _h_ is generated at time _bfttime_, then there is a set _C_ of validator pairs, such that _C_ is a subset of _NextValidators_ at height _h_, where:
  - The validator pairs in _C_ hold more than two-thirds of the total voting power in _NextValidators_ at height _h_
  - Every validator in _C_ follows *all* protocols until _bfttime + trustingPeriod_

<!-- Formally,
\[
\sum*{(v,p) \in h.Header.NextV \wedge correct(v,h.Header.bfttime + trustingPeriod)} p >
2/3 \sum*{(v,p) \in h.Header.NextV} p
\] -->

_Remark:_ The definition of correct [**[TMBC-CORRECT]**](TMBC-CORRECT-link) refers to realtime, while it is used here with _bfttime_ and _trustingPeriod_, which are "hardware times". 
Due to [**[TMBC-TIME]**](TMBC-TIME-link), we do not make a distinction here. 




## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

Each correct full node _p_ maintains its local copy of the Tendermint blockchain, denoted by _decision_p_. 

A block is a data structure described [here](block).

#### **[TMBC-D-VAL]**:
There is a predicate _Valid()_ defined over blocks.
(perhaps we do not need that now)

### Temporal Properties

> safety specifications / invariants in English

> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually.





### Safety

#### **[TMBC-VC_AGR]**:
It is always the case that, for any two correct full nodes _p_ and _q_, it holds that _decision_p_ is a prefix of _decision_q_ or _decision_q_ is a prefix of _decision_p_.

#### **[TMBC-VC_VAL]**:
TODO: Not clear. 
Application specific. 
We will need to fix that eventually. 
I guess for fastsync and light client we don't need it.
(It is always the case that each _decision_ list entry _b_ satisfies _Valid(b)_. not
  sure. _Valid_ may take the current state)
(_Remark:_ Validity should make reference to the mempool, e.g., only messages from the
  mempool + we will need a spec for the mempool. For now I leave it like that
  as the light client and fastsync do not care about that.).



### Liveness
#### **[TMBC-VC-PROG]**: 
For all correct full nodes _p_ and all times _t_ there exists a time _t'_, such that _|decision_p(t)| < |decision_p(t')|_.


> How is the problem statement linked to the "Sequential Problem statement".
Simulation, implementation, etc. relations

### Solving the sequential specification
TODO: How does the distributed specification map to the sequential one? The argument should arrive at:

#### **[TMBC-CorrFull]**: 
Every correct Tendermint full node locally stores a prefix of the current list of headers from [**[TMBC-SEQ]**](TMBC-SEQ-link).


## Definitions

> In this section we become more concrete, with basic (abstracted) data types

> some math that allows to write specifications and pseudo code solution below.
Some variables, etc.

### Data structures

The following are data structures that are needed for this specification.

```go
type Validator struct {
    Address       Address     
    VotingPower   int64       
}
```
The `Validator` data structure is used to capture the validator pair, 
defined in [**[TMBC-VALIDATOR]**](#tmbc-validator).
It has the fields
  - `Address`, an address of the validator node
  - `VotingPower`, an integer, denoting the validator's voting power 

---
```go
type ValidatorSet struct {
    Validators         []Validator
    TotalVotingPower   int64
}
```
The `ValidatorSet` data structure is used to model a set of `Validators`, and it has the following fields:
 - `Validators`, a collection of `Validator` data structures, 
 - `TotalVotingPower`, an integer, denoting the total voting power of all the validator pairs in `Validators`. 


---
```go
type Header struct {
    Height               int64
    Time                 Time          
    LastBlockID          BlockID       
    ValidatorsHash       []byte        
    NextValidatorsHash   []byte        
}
```
The `Header` data structure is essential for this specification. 
It contains the following fields, as defined in [**[TMBC-HEADER-FIELDS]**](#tmbc-header-fields):
  - `Height`, an integer, denoting the height of the header
  - `Time`, a time point, denoting the chain time when the header was generated
  - `LastBlockID`, a block idenitifier, which dis used as a pointer to the previous block
  - `ValidatorsHash`, a hash of the validator set for the current block
  - `NextValidatorsHash`, a hasn of the validator set for the next block
---
For the remaining data structures, we refer to [arXiv paper](arXiv).

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



# References

[[block]] Definition of the block data structure

[[blockchain]] Tendermint Blockcahin specification

[[fastsync]] Specification of the fastsync protocol

[[fullnode]] Specification of the full node API

[[header]] Definition of the header data structure

[[lightclient]] Light Client ADR

[[verifier]] Light Client Verification Specification

[[arXiv]] The Tendermint paper on arXiv


[block]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md#block 
[blockchain]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md#blockchain
[fastsync]: https://github.com/informalsystems/VDD/
[lightclient]: https://github.com/interchainio/tendermint-rs/blob/e2cb9aca0b95430fca2eac154edddc9588038982/docs/architecture/adr-002-lite-client.md#adr-002-light-client
[verifier]: https://github.com/informalsystems/VDD/blob/master/lightclient/verification.md#core-verification
[header]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md#header
[fullnode]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md

[TMBC-SEQ-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-seq
[TMBC-VALIDATOR-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-validator
[TMBC-CORRECT-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-correct
[TMBC-TIME-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-time

[arXiv]: https://arxiv.org/abs/1807.04938