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

#### **[TMBC-SEQ-APPEND-E]**: 
If a header is appended at time *t* then no additional header will be
appended before time *t + ETIME*


#### **[TMBC-SEQ-APPEND-L]**: 
If a header is appended at time *t* then the next header will be
appended before time *t + LTIME*

#### **[TMBC-SEQ-APPEND-ELEL]**: 
*ETIME <= LTIME*

*Remark:* *ETIME* and *LTIME* define the earliest and latest times at
which a new block is added. We might later parameterize by setting
*ETIME* to infinity, e.g., when we say that fastsync terminates in the
case the blockchain does not grow.



 
### Basic Soundness Conditions
 


#### **[TMBC-SOUND-INC-HEIGHT]**:
 For all *i < len(chain)*: *chain[i].Height + 1 = chain[i+1].Height*

*Remark:* We do not write *chain[i].Height = i* to allow that a chain
can be started at some arbitrary height, e.g., when the is social
consensus to restart a chain from a given height/block.

*Remark:* Was called [TMBC-SOUND-NO-HOLES]

#### **[TMBC-SOUND-INC-TIME]**:
 For all *i < len(chain)*: *chain[i].Time < chain[i+1].Time*


#### **[TMBC-SOUND-NextV]**:
For all *i < len(chain)*: *chain[i+1].Validators = chain[i].NextValidators*




###  Functions, Domains, and more soundness conditions

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
	 * *commit* is in PossibleCommit(b.Height), cf. [TMBC-SOUND-PossCommit].

*Remark.* Observe that *proof* refers to the *chain*. It thus depend on
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

#### **[TMBC-SOUND-CHAIN]**:
 For all *i < len(chain)*: *match-hash(chain[i], chain[i+1])*
 
#### **[TMBC-SOUND-LAST-COMM]*:
For all *i < len(chain)*: *match-proof(chain[i], chain[i+1])*


#### **[TMBC-SOUND-APP]**:
For all *i < len(chain)*: *chain[i+1].AppState = execute(chain[i].Data,chain[i].AppState)*


#### **[TMBC-SEQ-INV]**

At all times, the chain is sound [TMBC-SOUND-*].




 



 
# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly

> should have clear formalization in temporal logic.

## Timing, nodes, and correctness assumptions

In a Tendermint system, the nodes that choose to interact with each
other collectively implement the *chain* list in a distributed manner
by executing a protocol.  The Tendermint protocols should ensure that
the participating nodes cannot benefit by deviating from the expected
behavior.  As a result, the nodes should be motivated (incentivized)
to follow the protocol.  This is achieved by requiring the nodes to
bond atoms and by the system punishing the nodes that do not follow
the protocol.  That is, the nodes bond atoms in order to participate,
and they are punished if they misbehave.  If the nodes want to claim
their atoms, they have to wait for a certain period of time, called
the *unbonding period*.  The unbonding period is a configuration
parameter of the system.

#### **[TMBC-TIME-PARAMS]**:
A Tendermint blockchain has the following configuration parameters:
 - *unbondingPeriod*: a time duration.
 - *trustingPeriod*: a time duration smaller than *unbondingPeriod*.




#### **[TMBC-NODES]**:
Tendermint full nodes (or just *full nodes*), execute a set of
protocols, e.g., consensus, gossip, fast sync, etc.  When a full node
actively participates in the distributed consensus, it is called a
*validator node*.

#### **[TMBC-CORRECT]**:
We define a predicate *correct(n, t)*, where *n* is a node and *t* is a 
time point. 
The predicate *correct(n, t)* is true if and only if the node *n* 
follows all the protocols until time *t*.

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
The authenticated Byzantine model assumes [TMBC-Sign-NoForge] and
[TMBC-FaultyFull], that is, faulty nodes are limited in that they
cannot forge messages [TMBC-Sign-NoForge].




## Validators

In [TMBC-HEADER-Fields], most of the fields are defined for abstract
domains. Here we will specialize DomainVal and DomainCommit, and
describe how validators and commit are implemented in Tendermint
consensus.

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

#### **[TMBC-VP-SOUND-VALID-UNIQUE]**:
For each block in the chain, the set *Validators* contains at most 
one validator pair for each full node.

#### **[TMBC-VP-SOUND-NEXT-VALID-UNIQUE]**:
For each block in the chain, the set *NextValidators* contains at 
most one validator pair for each full node.
 


## Distributed Definition of Commit



#### **[TMBC-VOTE]**:
A vote contains a `prevote` or `precommit` message sent and signed by
a validator node during the execution of [consensus][arXiv]. Each 
message contain the following field
   - `Type`: prevote or precommit
   - `Height`
   - `Round` a positive integer
   - `BlockID` a Hashvalue of a block (not necessarily a block of the chain)


#### **[TMBC-COMMIT]**:
A commit is a set of signed votes.

#### **[TMBC-SOUND-DISTR-PossCommit]**:
For a block *b*, each element *pc* of *PossibleCommit(b)* satisfies:
  - each vote *v* in *pc* satisfies
     * *pc* contains only signed votes by validators from *b.Validators*
     * v.blockID - hash(b)
	 * v.Height = b.Height *TODO:* complete the checks here
  - the sum of the voting powers in *pc* is greater than 2/3
  TotalVotingPower(b.Validators) 


#### **[TMBC-SOUND-DISTR-LAST-COMM]**:
Combining the specialization of *PossibleCommit* from
[TMBC-SOUND-DISTR-PossCommit] with the abstract definitions
[TMBC-SOUND-FUNCTIONS] and [TMBC-SOUND-LAST-COMM] we obtain the
definition of soundness for LastCommit in the distributed setting.
	 





## Commit Invariants

Commit messages are used to establish proof that a certain block is on
the blockchain. 

We now make explicit some invariants a correct validator must ensure.
We consider a predicate `valid` over blocks, and `precommit`
     messages of the [consensus algorithm][arXiv].
Correct validators use *valid* to ensure the soundness requirements of
     the blockchain [TMBC-SOUND-?], and send *precommit* messages
     only for blocks for which *valid* evaluates to true.

#### **[TMBC-INV-CORR-PROC-VALID]**:

There is a predicate `valid` over a block (and the prefix of the
chain, which we omit in the notation for now).
In particular, if *valid(b)* evaluates to true at a correct validator,
     then *b.Validators = b'.NextValidators* of the block *b'* of
     height *h - 1* of the blockchain;


The following invariant is crucial to guarantee the soundness of the chain:


#### **[TMBC-INV-CORR-PROC-COMMIT]**:

A correct validator sends and signs precommit for a block *b*, only if `valid(b)`.

*Remark:*  Code line 36 in the  [consensus algorithm][arXiv].





From [TMBC-INV-CORR-PROC-VALID] and [TMBC-INV-CORR-PROC-COMMIT]
follows that **more than two thirds of the voting
power in *b.Validators* is correct for any block *b* signed by a correct
validator**. As a result, a commit that is well-formed (that is, is in
*PossibleCommit(b)*) and signed by a correct validator is a proof that
*b* is in the blockchain. 

"Signed by a correct validator" means that the validator *n*
sends *precommit* at time *t* and *correct(n, t)* holds.



#### **[TMBC-VAL-COMMIT]**:

If for a block *b*,  a commit *c*
  - contains one correct validator, 
  - is contained in *PossibleCommit(b)*
  
then the block *b* is on the blockchain.




## Tendermint failure model





#### **[TMBC-FM-2THIRDS]**:
(Tendermint Failure Model) If a block *h* is in the chain,
then there exists a subset *CorrV*
of *h.NextValidators*, such that:
  - *TotalVotingPower(CorrV) > 2/3
    TotalVotingPower(h.NextValidators)*; cf. [TMBC-VALIDATOR-Set]
  - For every validator pair *(n,p)* in *CorrV*, it holds *correct(n,
    h.Time + trustingPeriod)*; cf. [TMBC-CORRECT]



<!--- Formally,
\[
\sum*{(v,p) \in h.Header.NextV \wedge correct(v,h.Header.bfttime + trustingPeriod)} p >
2/3 \sum*{(v,p) \in h.Header.NextV} p
\] -->

*Remark:* The definition of correct
[**[TMBC-CORRECT]**](TMBC-CORRECT-link) refers to realtime, while it
is used here with *Time* and *trustingPeriod*, which are "hardware
times".  We do not make a distinction here.


From [TMBC-FM-2THIRDS] we directly derive the following observation:

#### **[TMBC-VAL-CONTAINS-CORR]**:

Given a (trusted) block *tb* of the blockchain, a set of full nodes 
*N* contains a correct node at a real-time *t*, if
   - *t - trustingPeriod < tb.Time < t*
   - the voting power in tb.NextValidators of nodes in *N* is more
     than 1/3 of *TotalVotingPower(tb.NextValidators)*





*Remark:* The light client verification checks [TMBC-VAL-CONTAINS-CORR] and
[TMBC-VAL-COMMIT] as follows: 
Given a trusted block *tb* and an untrusted block *ub* with a commit *cub*,
one has to check that *cub* is in *PossibleCommit(ub)*, and that *cub*
contains a correct node using *tb*.



Until now, we have established soundness of the blockchain, and some
invariants expected from correct validators when observed from the
outside. Below we describe the internals of the [consensus
algorithm][arXiv]. For details we refer to the [paper][arXiv] or to a
later (more complete version) of this specifications. *Remark:* right
now the goals is to have a formal understanding of the outside view of
the blockchain.

## Distributed Problem Statement

### Design choices


#### **[TMBC-FM-CONS]**: 
(Consensus failure model)
There is a set *C* of validator pairs, such that *C* is a subset of *NextValidators* at height *h*, where: 
  - The validator pairs in *C* hold more than two-thirds of the total voting power in *NextValidators* at height *h*
  - For every validator pair *(n,p)* in *C*, follows the consensus protocol until consensus for height *h+1* is terminated. 


We recall that [TMBC-CORRECT] denotes by *correct(n, t)* that full
node *n* is correct up to time *t* if it follows all the protocols
up to time *t*. For now we assume that both failure assumptions
[TMBC-FM-2THIRDS] and [TMBC-FM-CONS] hold.





Each correct full node *p* maintains its local copy of the Tendermint
blockchain, denoted by *chain_p*.



### Temporal Properties

> safety specifications / invariants in English

> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually.





### Safety

#### **[TMBC-VC_AGR]**:
At all times *t*, for any two full nodes *p* and *q*, with *correct(p,
     t)* and *correct (q, t)* it holds that *chain_p(t)* is a prefix
     of *chain_q(t)* or *chain_q(t)* is a prefix of *chain_p(t)*.

#### **[TMBC-VC_VAL]**:
For a full node *p*, we substitute *chain* with *chain_p* in the
soundness properties [TMBC-SOUND-?]. For all times *t* and every full
node *p*, with *correct(p, t)*, the soundness requirements hold for
*chain_p(t)*.


*Remark:* Validity should make reference to the mempool, e.g., only messages from the
  mempool + we will need a spec for the mempool. For now I leave it like that
  as the light client and fastsync do not care about that.
  
*Remark:* Additional application specific soundness requirements might
also need to hold.



### Liveness
#### **[TMBC-VC-PROG]**: 
For all correct full nodes *p* and all times *t* there exists a time
*t'*, such that *|chain_p(t)| < |chain_p(t')|*.



*Remark:* In the temporal properties above we use the *correct*
predicate, in a way that suggests that all full nodes participate in
Tendermint since the genesis block. However, there are Tendermint
protocols (state sync, fast sync) that allow nodes to join the system
later. We will have to define later what it means for these nodes to satisfy
[TMBC-VC_AGR] and [TMBC-VC_VAL] and [TMBC-VC-PROG]. For instance, they
may not need to have the complete prefix of *chain* but start at some height.


> How is the problem statement linked to the "Sequential Problem statement".
Simulation, implementation, etc. relations

### Solving the sequential specification

**TODO:** How does the distributed specification map to the sequential
one? For instance, at each time the longest prefix of *chain_p* for
some *p* defines *chain* in the sequential specification.

#### **[TMBC-CorrFull]**: 
Every correct Tendermint full node locally stores a prefix of the
current list of headers from [**[TMBC-SEQ]**](TMBC-SEQ-link).




**For the remainder, we refer to the [arXiv paper](arXiv) for now.**


## Definitions

> In this section we become more concrete, with basic (abstracted) data types

> some math that allows to write specifications and pseudo code solution below.
Some variables, etc.

### Data structures



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
