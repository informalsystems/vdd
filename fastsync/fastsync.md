
*** This is the beginning of an unfinished draft. Comments are welcome! ***

# Fastsync

> Rough outline of what the component is doing and why. 2-3 paragraphs 

# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc. 

> should give the reader the understanding in what environment this component
will be used. 

Fastsync is a protocol that is used by a full node to catchup to the
current state of a Tendermint blockchain. Its typical use case is a
full node that was disconnected from the system for some time. The
recovering full node locally has a copy of a prefix of the blockchain,
and the corresponding application state that is slightly outdated. It
then queries its peers for the blocks that were decided on by the
Tendermint blockchain during the period the full node was
disconnected. After receiving these blocks, it executes the
transactions in the blocks in order to catch-up to the current height
of the blockchain and the corresponding application state.

## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view. 

A full node has as input a block of the blockchain at height *h* and
the corresponding application state (or the prefix of the current
blockchain until height *h*). It has access to a set *peerIDs* of full
nodes called *peers* that it knows of.  The full node uses the peers
to read blocks of the Tendermint blockchain (in a safe way, that is,
it checks the soundness conditions), until it has read the most recent
block and then terminates.


## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

*Fastsync* gets as input a block of height *h* and the corresponding
application state *s*, and produces as output a list *l* of blocks
starting at height *h* to some height *terminationHeight*,
and the application state
when applying the transaction of the list *l* to *s*.

#### **[FS-Seq-Live]**: 
*Fastsync* eventually terminates.

 
#### **[FS-Seq-Term]**:
Let *bh* be the height of the blockchain at the time *Fastsync* starts.
When *Fastsync* terminates, it outputs a list of all blocks from
height *h* to some height *terminationHeight >= bh*,
[**[TMBC-SEQ]**](TMBC-SEQ-link).




#### **[FS-Seq-Inv]**:
Upon termination, the application state is the one that corresponds to
the blockchain at height *terminationHeight*.




# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects 

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly 

> should have clear formalization in temporal logic.

#### **[FS-A-NODE]**:
We consider a node *FS* that performs *Fastsync*.
It has access to a set *peerIDs* of IDs (public keys) of peers (full
     nodes).

#### **[FS-A-PEER]**:
Peers can be faulty, and we do not make any assumption about number or
ratio of correct/faulty nodes.

#### **[FS-A-VAL]**:
The system satisfies [TMBC-Auth-Byz] and [TMBC-FM-2THIRDS]. Thus, there is a
blockchain that satisfies the soundness requirements [TMBC-SOUND-?].

#### **[FS-A-COMM]**:
Communication between *FS* and all correct peers is reliable and
bounded in time: there is a message end-to-end delay *Delta* such that
if a message is sent at time *t* by a correct process to a correct process, then it will be received and
processes by time *t + Delta*. This implies that we need a timeout of
at least *2 Delta* for remote procedure calls to ensure that the
response of a correct peer arrives before the timeout expires.

#### **[FS-A-LCC]**:
The node *FS* executing Fastsync is following the protocol (it is correct).


## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

We do not put assumptions on the existence of a correct full node
in *peerIDs*. Under this assumption we cannot guarantee the properties
described in the sequential specification above. Thus, in the (unreliable)
distributed setting, we consider two kinds of termination (normal and abort):

#### **[FS-DISTR-TERM]**:
*Fastsync* may *terminate normally* or it  *aborts*.


#### Remote Functions

The Tendermint Full Node exposes the following functions over Tendermint RPC:

*Remark:* we will have asynchronous RPCs


```go
func Status(addr Address) (int64, error)
```
- Implementation remark
   - RPC to full node *addr*
- Expected precondition
  - none
- Expected postcondition
  - if *addr* is correct: Returns the current height `height` of the
    peer. [FS-A-COMM]
  - if *addr* is faulty: Returns an arbitrary height. [TMBC-Auth-Byz]
- Error condition
   * if *addr* is correct: none. By [FS-A-COMM] we assume communication is reliable and timely.
   * if *addr* is faulty: arbitrary error (including timeout). [TMBC-Auth-Byz]
----


 ```go    
func Block(addr Address, height int64) (Block, error)
```
- Implementation remark
   - RPC to full node *addr*
- Expected precodnition
  - header of `height` is less than or equal to height of the peer
- Expected postcondition
  - if *addr* is correct: Returns the block of height `height`
  from the blockchain. [FS-A-COMM]
  - if *addr* is faulty: Returns arbitrary block [TMBC-Auth-Byz]
- Error condition
  - if *addr* is correct: precondition violated. [FS-A-COMM]
  - if *addr* is faulty: arbitrary error (including timeout). [TMBC-Auth-Byz]
----

### Temporal Properties



We sometimes consider the following (fairness) constraints:

#### **[FS-CORR-PEER]**:
The set *peerID* contains one correct full node.





> safety specifications / invariants in English 



#### **[FS-VC-NONABORT]**:
Under [FS-CORR-PEER], *Fastsync* never aborts. (Together with
[FS-VC-TERM] below that means it will terminate normally.)


#### **[FS-VC-INV]**:
If *FastSync* terminates normally at height *terminationHeight*, then the
application state is the one that corresponds to the blockchain at
height *terminationHeight*.


#### **[FS-VC-CORR-INV]**:
Under [FS-CORR-PEER], let *t* be the maximum height of a correct peer [TMBC-CorrFull]
in *peerIDs* at the time *Fastsync* starts. If *FastSync* terminates
normally, it is at some height *terminationHeight >= t*.



> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually. 



#### **[FS-VC-TERM]**:
*Fastsync* eventually terminates normally or it eventually aborts.



> How is the problem statement linked to the "Sequential Problem statement". 
Simulation, implementation, etc. relations 



## Definitions

> In this section we become more concrete, with basic (abstracted) data types 

> some math that allows to write specifications and pseudo code solution below.
Some variables, etc. 


#### Fastsync has the following configuration parameters:
- *trustingPeriod*: a time duration
  [**[TMBC-TIME_PARAMS]**](TMBC-TIME_PARAMS-link).  
  **TODO:** Right now I largely ignore that. I put that into
  assumption [FS-A-INIT] below.  We should fix what we assume
  here, e.g., startBlock is "recent" and clock of fastsyncing node is
  synchronized to real-time.
  only approximately synchronized clocks.
  



#### Inputs
- *startBlock*: the block Fastsync starts from
- *startState*: application state corresponding to *startBlock.Height*

#### **[FS-A-INIT]**:
- *startBlock* is from the blockchain, and within *trustingPeriod*
(possible with some extra margin to ensure termination before
*trustingPeriod* expired)
- *startState* is the application state of the blockchain at Height *startBlock.Height*.

#### Variables
- *height*: initially *startBlock.Height*
- *state*: initially *startState*
- *peerIDs*: peer addresses 
- *peerHeigts*: stores for each peer the height it reported. initially 0
- *pendingBlocks*: stores for each height which peer was
  queried. initially nil for each height
- *receivedBlocks*: stores for each height which peer returned
  it. initially nil
- *blockstore*: stores for each height greater than
    *startBlock.Height*, the block of that height. initially nil for
    all heights

#### Auxiliary Function
- *TargetHeight = max {peerHeigts(addr): addr in peerIDs}*  


#### **[FS-VAR-STATE-INV]**:
It is always the case that the state corresponds to the state of the
blockchain of that height, that is, *state = chain[height].AppState*
[TMBC-SEQ]. 

#### **[FS-VAR-PEER-INV]**:
It is always the case that the set *peerIDs* only contains nodes that
have not yet misbehaved (by sending wrong data or timing out).



#### **[FS-VAR-PEER-INV]**:
If a peer never misbehaves, it is never removed from *peerIDs*. It
follows that under [FS-CORR-PEER], *peerIDs* is always non-empty.



## Solution

> Basic data structures. Simplified, so that we can focus on the distributed
algorithm here. If existing: link to Tendermint data structures, and mentioned
if details were omitted. 

### Outline

> Describe solution (in English), decomposition into functions, where communication to other components happens.

The protocol is described in terms of functions that are triggered by
(external) events:

- `QueryStatus()`: regularly (currently every 10sec; necessarily
  interval greater than *2 Delta*) queries all nodes from peerIDs set
  for their current height [TMBC-LOCAL-CHAIN]. It does so
  by calling `Status(n)` remotely on all peers *n*.
  
- `CreateRequest`: regularly checks whether certain blocks have no open
  request. If a block does not have an open request, it requests one from a full node. It does so by calling
  `Block(n,h)` remotely on one full node *n* for a missing height *h*.
  

The functions `Status` and `Block` are called by asynchronous
RPC. When they return, the following functions are called:

- `OnStatusResponse(addr Address, height int64)`: The full node with
  address *addr* returns its current height. The function updates the height
  information about *addr*, and may also increase *TargetHeight*.
  
- `OnBlockResponse(addr Address, b Block)`. The full node with
  address *addr* returns a block. It is added to *blockstore*. Then
  the auxiliary function `Execute` is called.

- `Execute()`: Iterates over the *blockstore*. Starts at height until
  the longest prefix. Checks soundness of block after block, and
  executes the transactions of a sound block and updates *state*. If
  the longest prefix reaches *TargetHeight* it terminates *Fastsync*.
  

If in the course of the execution *peerIDs* becomes empty (this is
only possible if [FS-CORR-PEER] is violated), then *FastSync* does **abort**.

### Details

> Function signatures followed by pseudocode (optional) and a list of features (required):
> - Implementation remarks (optional)
>   - e.g. (local/remote) function called in the body of this function
> - Expected precondition
> - Expected postcondition
> - Error condition

```go
func QueryStatus()
```
- Expected precondition
    - peerIDs initialized and non-empty
- Expected postcondition
    - call asynchronously `Status(n)` at each peer *n* in *peerIDs*.
- Error condition
    - fails if precondition is violated
----

```go
func OnStatusResponse(addr Address, height int64)
```
- Expected precondition
    - *peerHeights(addr) <= height*
- Expected postcondition
    - *peerHeights(addr) = height*
	- *TargetHeight* is updated
- Error condition
    - if precondition is violated: remove *addr* from *peerIDs*
- Timeout condition
    - if `OnStatusResponse(addr, h)` was not invoked within *2 Delta* after
	`Status(addr)`  was called: remove *addr* from *peerIDs*
----


```go
func CreateRequest
```
- Expected precondition
    - *height < TargetHeight*
	- *peerIDs* nonempty
- Expected postcondition
    - Function `Block` is called remotely at a peer *addr* in peerIDs 
	  for a missing height *h*  
	  *Remark:* different implementations may have different
      strategies to balance the load over the peers
    - *pendingblocks(h) = addr*
- Error condition
    - if *peerIDs* is empty: no correct peers left;  abort. Not
	possible under  [FS-CORR-PEER].
----


```go
func OnBlockResponse(addr Address, b Block)
```
- Expected precondition
    - *pendingblocks(b.Height) = addr*
	- *b* satisfies basic soundness  
	**TODO:** should this be checked here?
- Expected postcondition
    - it function `Execute` has been executed without error
    - *receivedBlocks(b.Height) = addr*
	- *blockstore(b.Height) = b*
- Error condition
    - if precondition is violated: remove *addr* from *peerIDs*; reset
	*pendingblocks(b.Height)* to nil;
- Timeout condition
    - if `OnBlockResponse(addr, b)` was not invoked within *2 Delta* after
	`Block(addr,h)` was called for *b.Height = h*: remove *addr* from *peerIDs*
----

```go
func Execute()
```
- Comments
    - none
- Expected precondition
    - [goodblocks]: *receivedBlocks* are all from the blockchain
	- application state is the one of the blockchain at height *height*
- Expected postcondition
    - height is updated height of complete prefix that matches the blockchain
	- state is the one of the blockchain at height *height*
	- if height = TargetHeight: **terminate normally**
- Error condition
    - if precondition [goodblocks] is violated: there is a bad block *b*; *b*
	removed from blockstore, node with Address
	receivedBlocks(b.Height) removed from peerIDs;
----












## Correctness arguments

> Proof sketches of why we believe the solution satisfies the problem statement.
Possibly giving inductive invariants that can be used to prove the specifications
of the problem statement 

# References

> links to other specifications/ADRs this document refers to


[block]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md

[blockchain]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md

[TMBC-HEADER-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-header

[TMBC-SEQ-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-seq

[TMBC-CorrFull-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-corrfull

[TMBC-Sign-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-sign

[TMBC-FaultyFull-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-faultyfull

[TMBC-TIME_PARAMS-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-time_params

[TMBC-FM-2THIRDS-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-fm-2thirds

[TMBC-INV-SIGN-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-inv-sign

[TMBC-INV-VALID-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-inv-valid

[LCV-VC-LIVE-link]: https://github.com/informalsystems/VDD/tree/master/lightclient/verification.md#lcv-vc-live

[lightclient]: https://github.com/interchainio/tendermint-rs/blob/e2cb9aca0b95430fca2eac154edddc9588038982/docs/architecture/adr-002-lite-client.md

[failuredetector]: https://github.com/informalsystems/VDD/blob/master/liteclient/failuredetector.md

[fullnode]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md

[FN-LuckyCase-link]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md#fn-luckycase

[blockchain-validator-set]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md#data-structures

[fullnode-data-structures]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md#data-structures

[FN-ManifestFaulty-link]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md#fn-manifestfaulty
