*** This is the beginning of an unfinished draft. Don't continue reading! ***

# Tendermint Blockchain

> Rough outline of what the component is doing and why. 2-3 paragraphs

A blockchain is a growing list of sets of transactions, denoted by *decision*. 
If several processes in a distributed system have access to the blockchain, they can (1) provide transactions as input and (2) use the decision list to execute these transactions in order. 

The *decision* list itself should be implemented in a reliable way, which introduces the need for fault-tolerance and distribution.
The Tendermint protocols implement the blockchain over an unreliable (Byzantine) distributed system. 
More precisely, a Tendermint system consists of many full nodes that each maintain a local copy of the (prefix of the current) *decision* list.

In this specification, we are only concerned with the *decision* lists. 
They are maintained at so-called [full nodes][fullnode], and are the result of the execution of the Tendermint consensus protocol (described in the ArXiv paper). 
The Tendermint consensus implements a loop with iterator *h* (height). 
In each iteration, upon deciding on the transactions to be put into the *decision* list, a new *decision* list entry is created.


# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc.

> should give the reader the understanding in what environment this component
will be used.

This specification is central in the collection of Tendermint protocols. 
The behavior of protocols like [fastsync][fastsync], or the [light client][lightclient] will be defined with respect to this specification. 
E.g., the light client implements a read operation of the *decision* list entry of some height *h*. 
It is thus crucial to understand what data is stored in this *decision* list entry, and what are the precise semantics of a read operation in a faulty environment.


## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view.

A blockchain provides a growing sequence of sets of transactions.

## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

#### **[TMBC-HEADER]**:
A set of blockchain transactions is stored in a data structure called *block*, which 
contains a field called *header*. (The data structure *block* is defined [here][block]). 
As the header contains hashes to the relevant fields of the block, for the purpose of this
specification, we will assume that the blockchain is a list of headers, rather than a
list of blocks. 
We also assume that every hash in the header identifies the data it hashes. 
Therefore, in this specification, we do not distinguish between hashes and the 
data they represent.

#### **[TMBC-SEQ]**:



The Tendermint blockchain is a list *chain* of headers. For all *i <=
len(chain)*, each header *chain[i]*
contains the following fields.

 - `Height`: non-negative integer
 - `Time`: time (integer)
 - `LastBlockID`: Hashvalue
 - `LastCommit` DomainCommit
 - `Validators`: DomainVal
 - `NextValidators`: DomainVal
 - `Data`: DomainTX
 - `AppState`: DomainApp
 - `LastResults`: DomainRes
 
 


 
### Invariants
 
#### **[TMBC-SEQ-INV-NOHOLES]**:
  If the blockchain contains a header of height *h*, then for all *h'
  < h*
it contains a header of height *h'*.
 

#### **[TMBC-SEQ-INV-INC]**:
 For all *i < len(chain)*: *chain[i].Height < chain[i+1].Height*

#### **[TMBC-SEQ-INV-TIME]**:
 For all *i < len(chain)*: *chain[i].Time < chain[i+1].Time*


#### **[TMBC-SEQ-INV-NV]**:
For all *i < len(chain)*: *chain[i+1].Validators = chain[i].NextValidators*


###  Functions, Domains, and more invariants

#### **[TMBC-SEQ-PossCommit]**:
There is a function PossibleCommit that maps for *0 < i <= len(chain)*, 
chain[i-1] to a set of values in DomainCommit.

*Remark.* TODO: in the distributed part we assume that no-one can
compute or guess that.




#### **[TMBC-SEQ-FUNCTIONS]**:
The system provides the following functions:

- `hash`: assumed to be a bijection
- `proof(b,commit)`: a predicate: true iff 
     * *b* is part of the chain
	 * proof is in PossibleCommit(b.Height)
- `skip-proof`: a predicate for blocks



The application provides the following function:

- `execute`: used for state machine replication. maps Data
  (transactions) and a state to a new state. It is a function
  (deterministic transitions)
  

Given two blocks *b* and *b'*:

- *match-hash(b,b') iff hash(b) = b'.LastBlockID*
- *match-proof(b,b') iff proof(b, b'.LastCommit)*

#### **[TMBC-SEQ-INV-BC]**:
 For all *i < len(chain)*: *match-hash(chain[i], chain[i+1])*
 
#### **[TMBC-SEQ-INV-LC]**:
For all *i < len(chain)*: *proof(chain[i], chain[i+1])*



#### **[TMBC-SEQ-INV-APP]**:
For all *i < len(chain)*: *chain[i+1].AppState = execute(chain[i].Data,chain[i].AppState)*

### Validation

*Remark:* The following formalizes block validation.
If I know one of them is from the blockchain, then the other
one is not.

#### **[TMBC-SEQ-VAL-BC]**:
Given two blocks *b* and *b'*, if *match-hash(b,b') = false*, then *b*
and *b'* are not subsequent headers of the blockchain.


#### **[TMBC-SEQ-VAL-LC]**:
Given two blocks *b* and *b'*, if *proof(b,b') = false*, then *b*
and *b'* are not subsequent headers of the blockchain.

#### **[TMBC-SEQ-SKIPVAL-LC]**:
Given two blocks *b* and *b'*, if *skip-proof(b,b') = true* and *b* is
a header in the blockchain, then *b'* is a header in the blockchain.


### Appending a block

#### **[TMBC-SEQ-GROW]**: 
During operation, new headers may be appended to the list one by one.

#### **[TMBC-SEQ-ASS-E]**: 
If a header is appended at time *t* then no additional header will be
appended before time *t + ETIME*


#### **[TMBC-SEQ-ASS-L]**: 
If a header is appended at time *t* then the next header will be
appended before time *t + LTIME*

#### **[TMBC-SEQ-ASS-ELEL]**: 
*ETIME <= LTIME*

*Remark:* *ETIME* and *LTIME* define the earliest and latest times at
which a new block is added. We might later parameterize by setting
*ETIME* to infinity, e.g., when we say that fastsync terminates in the
case the blockchain does not grow.



 
# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly

> should have clear formalization in temporal logic.

## Timing, nodes, and correctness assumptions

In a Tendermint system, the nodes that choose to interact with each other collectively 
implement the *decision* list in a distributed manner by executing a protocol. 
The Tendermint protocols should ensure that the participating nodes cannot benefit by 
deviating from the expected behavior. 
As a result, the nodes should be motivated (incentivized) to follow the protocol. 
This is achieved by requiring the nodes to bond atoms and by the system punishing the nodes that do not follow the protocol. 
That is, the nodes bond atoms in order to participate, and they are punished if they misbehave. 
If the nodes want to claim their atoms, they have to wait for a certain period of time, 
called the *unbonding period*. 
The unbonding period is a configuration parameter of the system. 

#### **[TMBC-TIME_PARAMS]**:
A Tendermint blockchain has the following configuration parameters:
 - *unbondingPeriod*: a time duration.
 - *trustingPeriod*: a time duration smaller than *unbondingPeriod*.

#### **[TMBC-NODES]**:
Tendermint full nodes (or just *full nodes*), execute a set of protocols, e.g., consensus, gossip, fast sync, etc. 
When a full node actively participates in the distributed consensus, it is called a *validator node*.

#### **[TMBC-CORRECT]**:
We define a predicate *correct(n, t)*, where *n* is a node and *t* is a 
time point. 
The predicate *correct(n, t)* is true if and only if the node *n* follows all the protocols until time *t*.

### Full node invariants

#### **[TMBC-FaultyFull]**:
No assumption is made about the behavior of faulty full nodes; they may be Byzantine.

#### **[TMBC-Sign]**:
Signatures and hashes cannot be broken.


## Blockchain data structure

#### **[TMBC-VALIDATOR]**:
Given a full node, a *validator* is a pair *(public key, voting power)*, where 
  - *public key* is the public key of the full node, 
  - *voting power* is an integer (representing the full node's voting power in a certain consensus instance).

Remark: We observed that the term *validator* refers to both a data structure and a full node that participates in the distributed computation. Therefore, we introduce the notions *validator pair* and *validator node*, respectively.

#### **[TMBC-HEADER-FIELDS]**:
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


```go
type Header struct {
	// basic block info
	Version  Version
	ChainID  string
	Height   int64
	Time     Time

	// prev block info
	LastBlockID BlockID

	// hashes of block data
	LastCommitHash []byte // commit from validators from the last block
	DataHash       []byte // MerkleRoot of transaction hashes

	// hashes from the app output from the prev block
	ValidatorsHash     []byte // validators for the current block
	NextValidatorsHash []byte // validators for the next block
	ConsensusHash      []byte // consensus params for current block
	AppHash            []byte // state after txs from the previous block
	LastResultsHash    []byte // root hash of all results from the txs from the previous block

	// consensus info
	EvidenceHash    []byte // evidence included in the block
	ProposerAddress []byte // original proposer of the block
```
The `Header` data structure is essential for this specification. 
  - `Height`, an integer, denoting the height of the header
  - `Time`, a time point, denoting the chain time when the header was generated
  - `LastBlockID`, a block idenitifier, which dis used as a pointer to the previous block
  - `ValidatorsHash`, a hash of the validator set for the current block
      *  *Validators*: a set of validator pairs (as defined in [**[TMBC-VALIDATOR]**](TMBC-VALIDATOR-link), of the validator nodes that participated in the consensus instance which generated the block,
  - `NextValidatorsHash`, a hasn of the validator set for the next
    block
     * *NextValidators*: a set of validator pairs of the full nodes
        that participate in the consensus instance of the next block,
 - *LastCommit*: the set of signatures of the validators that committed the last block.
---



#### **[TMBC-TIME]**:

TODO: fix after Zarko's comments

The time *bfttime* corresponds to the reading of the local clock of a validator (how this time is computed may change when the Tendermint consensus is modified). 
In this specification, we assume that all clocks are synchronized to real-time (and so is bfttime). 
We can make this more precise eventually (incorporating clock drift, accuracy, precision, etc.). 
Right now, we consider this assumption sufficient, as clock synchronization (under NTP) is in the order of milliseconds and *trustingPeriod* is in the order of weeks.

### Blockchain data

#### **[TMBC-DAT-DATA]**:
`Data` is a list of transactions

#### **[TMBC-DAT-COMMIT]**:
List of signatures





### Blockchain data structure invariants

#### **[TMBC-INV-SIGN]**:
The *LastCommit* field of the block at height *h+1* contains only signatures of 
validators. The validators that signed have more than two-thirds of the voting power at height *h*.

#### **[TMBC-INV-VALID]**:
The *NextValidator* set of a block at height *h* is equal to 
the *Validator* set 
of the block at height *h+1*.

#### **[TMBC-INV-VALID-UNIQUE]**:
The set *Validators* contains at most one validator pair for each full node.

#### **[TMBC-INV-NEXT-VALID-UNIQUE]**:
The set *NextValidators* contains at most one validator pair for each full node.
 
#### **[TMBC-INV-BLOCKID]**:
TODO: `BlockID` is a unique identifier of the block. (It contains the hash
(Merkleroot) of the fields in the header)
 
#### **[TMBC-INV-COMMIT]**:
TODO: what is to say about signatures. Votes? subset of validator of
previous block

## Failure model

#### **[TMBC-FM-CONS]**: 
(Consensus failure model)
There is a set *C* of validator pairs, such that *C* is a subset of *NextValidators* at height *h*, where: 
  - The validator pairs in *C* hold more than two-thirds of the total voting power in *NextValidators* at height *h*
  - Every validator pair in *C* follows the consensus protocol until consensus for height *h+1* is terminated. 



#### **[TMBC-FM-2THIRDS]**:
(Tendermint Failure Model)
If a block *h* is generated at time *bfttime*, then there is a set *C* of validator pairs, such that *C* is a subset of *NextValidators* at height *h*, where:
  - The validator pairs in *C* hold more than two-thirds of the total voting power in *NextValidators* at height *h*
  - Every validator in *C* follows *all* protocols until *bfttime + trustingPeriod*

<!-- Formally,
\[
\sum*{(v,p) \in h.Header.NextV \wedge correct(v,h.Header.bfttime + trustingPeriod)} p >
2/3 \sum*{(v,p) \in h.Header.NextV} p
\] -->

*Remark:* The definition of correct [**[TMBC-CORRECT]**](TMBC-CORRECT-link) refers to realtime, while it is used here with *bfttime* and *trustingPeriod*, which are "hardware times". 
Due to [**[TMBC-TIME]**](TMBC-TIME-link), we do not make a distinction here. 




## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

Each correct full node *p* maintains its local copy of the Tendermint blockchain, denoted by *decision_p*. 

A block is a data structure described [here](block).

#### **[TMBC-D-VAL]**:
There is a predicate *Valid()* defined over blocks.
(perhaps we do not need that now)

### Temporal Properties

> safety specifications / invariants in English

> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually.





### Safety

#### **[TMBC-VC_AGR]**:
It is always the case that, for any two correct full nodes *p* and *q*, it holds that *decision_p* is a prefix of *decision_q* or *decision_q* is a prefix of *decision_p*.

#### **[TMBC-VC_VAL]**:
TODO: Not clear. 
Application specific. 
We will need to fix that eventually. 
I guess for fastsync and light client we don't need it.
(It is always the case that each *decision* list entry *b* satisfies *Valid(b)*. not
  sure. *Valid* may take the current state)
(*Remark:* Validity should make reference to the mempool, e.g., only messages from the
  mempool + we will need a spec for the mempool. For now I leave it like that
  as the light client and fastsync do not care about that.).



### Liveness
#### **[TMBC-VC-PROG]**: 
For all correct full nodes *p* and all times *t* there exists a time *t'*, such that *|decision_p(t)| < |decision_p(t')|*.


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
