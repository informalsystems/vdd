*** This is the beginning of an unfinished draft. Don't continue reading! ***

# Tendermint Blockchain

> Rough outline of what the component is doing and why. 2-3 paragraphs

A blockchain is a growing list of sets of transactions, denoted by *chain*. 
If several processes in a distributed system have access to the blockchain, they can (1) provide transactions as input and (2) use the chain list to execute these transactions in order. 

The *chain* list itself should be implemented in a reliable way, which introduces the need for fault-tolerance and distribution.
The Tendermint protocols implement the blockchain over an unreliable (Byzantine) distributed system. 
More precisely, a Tendermint system consists of many full nodes that each maintain a local copy of the (prefix of the current) *chain* list.

In this specification, we are only concerned with the *chain* lists. 
They are maintained at so-called [full nodes][fullnode], and are the
result of the execution of the Tendermint consensus protocol,
described in the [ArXiv paper][arXiv], and are called *decision* in
this paper. 
The Tendermint consensus implements a loop with iterator *h* (height). 
In each iteration, upon deciding on the transactions to be put into the *chain* list, a new *chain* list entry is created.


# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc.

> should give the reader the understanding in what environment this component
will be used.

This specification is central in the collection of Tendermint protocols. 
The behavior of protocols like [fastsync][fastsync], or the [light client][lightclient] will be defined with respect to this specification. 
E.g., the light client implements a read operation of the *chain* list entry of some height *h*. 
It is thus crucial to understand what data is stored in this *chain* list entry, and what are the precise semantics of a read operation in a faulty environment.


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

#### **[TMBC-HASH-UNIQUENESS]**:
We assume that every hash in the header identifies the data it hashes. 
Therefore, in this specification, we do not distinguish between hashes and the 
data they represent.


#### **[TMBC-HEADER-Fields]**:
A header contains the following fields:

 - `Height`: non-negative integer
 - `Time`: time (integer)
 - `LastBlockID`: Hashvalue
 - `LastCommit` DomainCommit
 - `Validators`: DomainVal
 - `NextValidators`: DomainVal
 - `Data`: DomainTX
 - `AppState`: DomainApp
 - `LastResults`: DomainRes


#### **[TMBC-SEQ]**:
#### **[TMBC-SEQ-LIST]**:

The Tendermint blockchain is a list *chain* of headers. 

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




 
### Basic Validity Conditions
 
#### **[TMBC-SOUND-NOHOLES]**:
  If the blockchain contains a header of height *h*, then for all *h'
  < h*
it contains a header of height *h'*.
 

#### **[TMBC-SOUND-INC]**:
 For all *i < len(chain)*: *chain[i].Height + 1 = chain[i+1].Height*

#### **[TMBC-SOUND-TIME]**:
 For all *i < len(chain)*: *chain[i].Time < chain[i+1].Time*


#### **[TMBC-SOUND-NV]**:
For all *i < len(chain)*: *chain[i+1].Validators = chain[i].NextValidators*


###  Functions, Domains, and more invariants

#### **[TMBC-SOUND-PossCommit]**:
There is a function PossibleCommit that maps a block (header) to a set
of values  in DomainCommit.



<!---
There is a function PossibleCommit that maps for *0 < i <= len(chain)*, 
chain[i-1] to a set of values in DomainCommit.
-->


#### **[TMBC-SOUND-FUNCTIONS]**:
The system provides the following functions:

- `hash`: We assume that every hash identifies the data it hashes

- `execute`: used for state machine replication. maps Data
  (transactions) and a state to a new state. It is a function
  (deterministic transitions).  
  **TODO:** it is provided by the
  application. Do we need to talk about the application in this spec?

- `proof(b,commit)`: a predicate: true iff 
     * *b* is part of the *chain*
	 * proof is in PossibleCommit(b.Height)

*Remark.* Observe that *proof* refers to the *chain*. It thus depends on
the execution, which changes quantification. For instance, we say
"there exists a function *hash* such that for all runs", while we
say "for each run there exists a function *proof*". The consequence is
that *hash* is a predetermined function (implemented), while *proof*
will have to be computed during the run as a function of the
*chain*. The challenge in a distributed system is to locally compute
*proof* without necessarily having complete knowledge of *chain*. In
the context of the light client, we even want to infer knowledge about
*chain* from the outcomes of the local computation of *proof*. We will
use digital signatures for that.



Given two blocks *b* and *b'*:

- `match-hash(b,b')` iff *hash(b) = b'.LastBlockID*
- `match-proof(b,b')` iff *proof(b, b'.LastCommit)*

#### **[TMBC-SOUND-BC]**:
 For all *i < len(chain)*: *match-hash(chain[i], chain[i+1])*
 
#### **[TMBC-SOUND-LC]**:
For all *i < len(chain)*: *match-proof(chain[i], chain[i+1])*



#### **[TMBC-SOUND-APP]**:
For all *i < len(chain)*: *chain[i+1].AppState = execute(chain[i].Data,chain[i].AppState)*


#### **[TMBC-SEQ-INV]**

At all times, the chain is sound [TMBC-SOUND-*].

### Validation


 



 
# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly

> should have clear formalization in temporal logic.

## Timing, nodes, and correctness assumptions

In a Tendermint system, the nodes that choose to interact with each other collectively 
implement the *chain* list in a distributed manner by executing a protocol. 
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

## Authenticated Byzantine Model

Due to the incentives, we claim that it is safe to assume below that
most of the full nodes will follow the protocol. However, we also have
to capture that full nodes deviate from the prescribed behavior,
either due to misconfiguration or implementation bugs, or because of
adversarial behavior. At the same time, we will heavily use digital
signature, and they constitute a stable technology that allows to
determine the sender of a message, even if a messages is wrapped into
another message and forwarded. In the distributed algorithm literature
(e.g., [[DLS88]][DLS]), this model is called **authenticated
Byzantine**.  Similar to Nancy Lynch when writing her book on
distributed algorithms, "we do not know of a nice formal definition"
for Byzantine failures with authentication.  So, let's try something
that we can at least use for verification.

#### **[TMBC-Sign]**:
- For each node with address *addr*, there is a function *sign_addr*
  that maps *Data* to a domain *SignedData(addr)*.
- SignedData is the disjoint union of *SignedData(addr)* over all *addr*.
- There is a function *check* that maps SignedData *sd*
  to the pair *(data, addr)* such that *sd = sign_addr(data)*.

#### **[TMBC-Sign-NoForge]**:
For all runs *r*, for all nodes *p* and *q* with address *aq*, for all
*sd* from SignedData, if *p*
sends a message that contains *sd* from *SignedData(aq)* in run *r*,
then *q* has sent a message containing *sd* earlier in run *r*.

*Remark:* [TMBC-Sign-NoForge] can be written as invariant over the
message history.


#### **[TMBC-FaultyFull]**:
No assumption is made about the internal
behavior of faulty full nodes.

#### **[TMBC-Auth-Byz]**:
[TMBC-Sign-NoForge] and [TMBC-FaultyFull].

*Remark:* [TMBC-Sign-NoForge]


## Validators

In [TMBC-HEADER-Fields], most of the fields are defined for abstract
domains. Here we will specialize DomainVal and DomainCommit, and
describe how **TODO:** commit and others  
are implemented in Tendermint.

*Remark:* We observed that in the existing documentation the term
*validator* refers to both a data structure and a full node that
participates in the distributed computation. Therefore, we introduce
the notions *validator pair* and *validator node*, respectively, to
distinguish these notions in the cases where they are not clear from
the context.


#### **[TMBC-VALIDATOR-Pair]**:

Given a full node, a 
*validator pair* is a pair *(public key, voting power)*, where 
  - *Address* is the address (public key) of the full node, 
  - *voting power* is an integer (representing the full node's
  voting power in a certain consensus instance).
  
*Remark:* In the Golang implementation the data type for *validator
pair* is called `Validator`


#### **[TMBC-VALIDATOR-Set]**:

A *validator set* is a set of validator pairs. For a validator set
*vs*, we write TotalVotingPower(vs) for the sum of the voting powers
of its validator pairs.

**TODO:** Commit - *LastCommit*: the set of signatures of the
validators that committed the last block.





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



### Validation under [TMBC-FM-2THIRDS]

#### **[TMBC-INV-COMMIT]**:

TODO: formalize A correct node does not sign headers off chain.




#### **[TMBC-VAL-COMMIT]**:

TODO: If a commit contains one correct validator, then the commit is
in Domaincommit. 

We have to make sure that a set of processes  contains a correct validator

#### **[TMBC-VAL-CONTAINS-CORR]**:

Given a set of full nodes *N*, a real-time *t*, a block *tb*, 
*Est-Trust-At(tb,N,t)* is true if
   - *tb.Time > t - trustingPeriod*
   - the voting power of nodes in *N* in tb.NextValidators is more
     than 1/3


#### **[TMBC-VAL-SKIP]**

Given two blocks *tb* and *b* and a *commit* and a real-time *t*, 
 *skip-proof(tb,b,bcommit,t) = true* if
   - *proof(b,bcommit) = true*
   - *Est-Trust-At(tb,bcommit,t)*
   - tb.Height < b.Height

#### **[TMBC-VAL-VERIF]**
Given two blocks *tb* and *b* and a *commit*,  
   - *tb* is a header in the blockchain,
   - at real-time *now*, the predicate *skip-proof(tb,b,bcommit,now)*
     evaluates to *true*
 
then *b* is from the blockchain.




### Refinements of Validation (from above)



**TODO:** refine validation from above for time


## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

Each correct full node *p* maintains its local copy of the Tendermint blockchain, denoted by *chain_p*. 

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
It is always the case that, for any two correct full nodes *p* and *q*, it holds that *chain_p* is a prefix of *chain_q* or *chain_q* is a prefix of *chain_p*.

#### **[TMBC-VC_VAL]**:
TODO: Not clear. 
Application specific. 
We will need to fix that eventually. 
I guess for fastsync and light client we don't need it.
(It is always the case that each *chain* list entry *b* satisfies *Valid(b)*. not
  sure. *Valid* may take the current state)
(*Remark:* Validity should make reference to the mempool, e.g., only messages from the
  mempool + we will need a spec for the mempool. For now I leave it like that
  as the light client and fastsync do not care about that.).



### Liveness
#### **[TMBC-VC-PROG]**: 
For all correct full nodes *p* and all times *t* there exists a time *t'*, such that *|chain_p(t)| < |chain_p(t')|*.


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
[fastsync]: https://github.com/informalsystems/VDD/blob/master/fastsync/fastsync.md
[lightclient]: https://github.com/interchainio/tendermint-rs/blob/e2cb9aca0b95430fca2eac154edddc9588038982/docs/architecture/adr-002-lite-client.md#adr-002-light-client
[verifier]: https://github.com/informalsystems/VDD/blob/master/lightclient/verification.md#core-verification
[header]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md#header
[fullnode]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md

[TMBC-SEQ-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-seq
[TMBC-VALIDATOR-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-validator
[TMBC-CORRECT-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-correct
[TMBC-TIME-link]: https://github.com/informalsystems/VDD/blob/master/lightclient/blockchain.md#tmbc-time

[arXiv]: https://arxiv.org/abs/1807.04938

[DLS]: https://groups.csail.mit.edu/tds/papers/Lynch/jacm88.pdf
