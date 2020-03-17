
*** This is the beginning of an unfinished draft. Don't continue reading! ***

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
transactions in the block in order to catch-up to the current height
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
starting at height *h* to some height *t*, and the application state
when applying the transaction of the list *l* to *s*.

#### **[FS-Seq-Live]**: 
*Fastsync* eventually terminates.

*Remark:* this will require timing assumptions on the rate at which a
block is added to the blockchain, and message and computation delays
involved in Fastsync. That is, the time it takes fastsync to append a
block to the list, and do the verification and the computation of the
application state should be less than *ETIME* from [TMBC-SEQ-APPEND-E].
 
#### **[FS-Seq-Term]**:
Let *bh* be the height of the blockchain at the time *Fastsync* starts.
When *Fastsync* terminates, it outputs a list of all blocks from
height *h* to some height *t >= bh*,
[**[TMBC-SEQ]**](TMBC-SEQ-link).




#### **[FS-Seq-Inv]**:
Upon termination, the application state is the one that corresponds to
the blockchain at height *t*.




# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects 

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly 

> should have clear formalization in temporal logic.

*Fastsync* has access to a list of peers (full nodes) called *peerIDs*

#### **[FS-A-PEER]**:
Peers can be faulty, and we do not make any assumption about number or
ratio of correct/faulty nodes.

#### **[FS-A-VAL]**:
The system satisfies [TMBC-Auth-Byz] and [TMBC-FM-2THIRDS]. Thus, there is a
blockchain that satisfies the soundness requirements [TMBC-SOUND-?].

#### **[FS-A-COMM]**:
Communication between Fastsync and a correct peer is reliable and
bounded in time.

#### **[FS-A-LCC]**:
The node executing Fastsync is following the protocol (it is correct).


## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

As we do not put assumptions on the existence of a correct full node
in *peerIDs*. Under this assumption we cannot guarantee the properties
described in the sequential specification above. Thus, in the
distributed setting, we consider two kinds of termination:

#### **[FS-DISTR-TERM]**:
*Fastsync* may terminate normally or aborts.


The Tendermint Full Node exposes the following functions over Tendermint RPC:

*Remark:* we will have asynchronous RPCs

*Remark:* This will go to full node spec.

```go
func Status(addr Address) (int64, error)
```
- Implementation remark
   - RPC to full node *n*
- Expected precodnition
  - none
- Expected postcondition
  - if *n* is correct: Returns the current height `height` of the peer
   if communication is timely (no timeout)
  - if *n* is faulty: Returns an arbitrary height
- Error condition
   * if *n* is correct: timeout
   * if *n* is faulty: arbitrary error
----


 ```go    
func Block(addr Address, height int64) (Block, error)
```
- Implementation remark
   - RPC to full node *n*
- Expected precodnition
  - header of `height` is less than or equal to height of the peer
- Expected postcondition
  - if *n* is correct: Returns the block of height `height`
  from the blockchain if communication is timely (no timeout)
  - if *n* is faulty: Returns arbitrary block
- Error condition
  - if *n* is correct: precondition violated or timeout 
  - if *n* is faulty: arbitrary error
----

### Temporal Properties



We sometimes consider the following (fairness) constraints:

#### **[FS-CORR-PEER]**:
The set *peerID* contains one correct full node.

#### **[FS-LUCKY-CASE]**:
The peer on which the RPC is called is correct and no timeout occurs
at the caller. 




> safety specifications / invariants in English 



#### **[FS-VC-NONABORT]**:
Under [FS-CORR-PEER] and [FS-LUCKY-CASE], *Fastsync* never aborts.

*Remark:* together with [FS-VC-TERM] below that means it will
terminate normally.


#### **[FS-VC-INV]**:
If *FastSync* terminates normally at height *t*, then the
application state is the one that corresponds to the blockchain at
height *t*.


#### **[FS-VC-CORR-INV]**:
Under [FS-CORR-PEER] and [FS-LUCKY-CASE], let *t* be the maximum
height of a correct peer in *peerIDs* at the time *Fastsync*
starts. If *FastSync* terminates normally, it is at some height *t' >=
t*.



> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually. 



#### **[FS-VC-TERM]**:
*Fastsync* eventually terminates normally or it eventually aborts.

#### **[FS-VC-NONAB]**:
Under [FS-CORR-PEER] and [FS-LUCKY-CASE], *Fastsync* does not abort.

> How is the problem statement linked to the "Sequential Problem statement". 
Simulation, implementation, etc. relations 

## Definitions

> In this section we become more concrete, with basic (abstracted) data types 

> some math that allows to write specifications and pseudo code solution below.
Some variables, etc. 


#### Fastsync has the following configuration parameters:
- *trustingPeriod*: a time duration [**[TMBC-TIME_PARAMS]**](TMBC-TIME_PARAMS-link).
- *clockDrift*: a time duration. Correction parameter dealing with
  only approximately synchronized clocks.
  



#### Inputs
- *startBlock*: block of height Fastsync starts from
- *startState*: state corresponding to startBlock.Height

#### Variables
- *height*: initially *startBlock.Height*
- *state*: initially *startState*
- *peerIDs*: peer addresses 
- *peerHeigts*: stores for each peer the height it reported. initially 0
- *pendingBlocks*: stores for each height which peer was
  queried. initially nil
- *receivedBlocks*: stores for each height which peer returned it
- *blockstore*: stores for each height greater than
    *startBlock.Height*, the block of that height

#### Macro
- *TargetHeight = max {peerHeigts(addr): addr in peerIDs}*  
  *Remark:* it is only computed over peers that are not considered
  faulty, yet

## Solution

> Basic data structures. Simplified, so that we can focus on the distributed
algorithm here. If existing: link to Tendermint data structures, and mentioned
if details were omitted. 

### Outline

> Describe solution (in English), decomposition into functions, where communication to other components happens.

The protocols is described in terms of functions that are triggered by
(external) events:

- `QueryStatus()`: regularly (every 10sec) queries all known full nodes
  for their current height of their local chain [addlink]. It does so
  by calling `Status(n)` remotely on all known full nodes *n*.
  
- `CreateRequest`: regularly checks whether certain blocks have no open
  request and queries one from a full node. It does so by calling
  `Block(n,h)` remotely on one full node *n* for a missing height *h*.
  

The functions `Status` and `Block` are called by asynchronous
RPC. When they return, the following functions are called:

- `OnStatusResponse(addr Address, height int64)`: The full node with
  address *addr* returns its current height. This updates the height
  information about *addr*, and may also increase *TargetHeight*
  
- `OnBlockResponse(addr Address, b Block)`. The full node with
  address *addr* returns a block. It is added to *blockstore*. Then
  the auxiliary function `Execute` is called.

- `Execute()`: Iterates over the *blockstore*. Starts at height until
  the longest prefix. Checks soundness of
  block after block, and executes the transactions of a sound
  block. If the longest prefix reaches *TargetHeight* it terminates
  fastsync.
  
### Function definitions
 
```go
func QueryStatus() {
  for i, s range peerIDs {
    call asynchronously Status(s)
  }
}
```
- Comments
    - 
- Expected precondition
    - peerIDs initialized and non-empty
- Expected postcondition
    - 
- Error condition
    - fails if precondition is violated

----

```go
func OnStatusResponse(addr Address, height int64)
```
- Comments
    - 
- Expected precondition
    - *peerHeights(addr) <= height*  
	  **TODO:** If messages can be re-ordered this precondition should
      be removed.
- Expected postcondition
    - *peerHeights(addr) = height*
	- (*Remark:* *TargetHeight* is updated)
- Error condition
    - if precondition is violated: remove *addr* from *peerIDs*
----


```go
func CreateRequest
```
- Comments
    - calls function `Block` remotely (asynchronously)
- Expected precondition
    - *height < TargetHeight*
	- *peerIDs* nonempty
- Expected postcondition
    - Function `Block` is called remotely at a peer *addr* in peerIDs 
	  for a missing height  
	  *Remark:* different implementations may have different
      strategies to balance the load over the peers
    - *pendingblocks(b.Height) = addr*
- Error condition
    - if *peerIDs* is empty: no correct peers left;  abort.
----


```go
func OnBlockResponse(addr Address, b Block)
```
- Comments
    - it calls `Execute`
- Expected precondition
    - *pendingblocks(b.Height) = addr*
	- *b* satisfies basic soundness?
- Expected postcondition
    - *receivedBlocks(b.Height) = addr*
	- *blockstore(b.Height) = b*
- Error condition
    - if precondition is violated: remove *addr* from *peerIDs*; reset
	*pendingblocks(b.Height) to nil;
----

TODO next:
```go
func Execute()
```
- Comments
    - none
- Expected precondition
    - [b] *receivedBlocks* are all from the blockchain
	- state is the one of the blockchain at height *height*
- Expected postcondition
    - height is updated height of complete prefix that matches the blockchain
	- state is the one of the blockchain at height *height*
	- if height = TargetHeight: stop everything
- Error condition
    - if precondition [b] is violated: there is a bad block *b*; *b*
	removed from blockstore, node with Address
	receivedBlocks(b.Height) removed from peerIDs
----







### Details

> Function signatures followed by pseudocode (optional) and a list of features (required):
> - Implementation remarks (optional)
>   - e.g. (local/remote) function called in the body of this function
> - Expected precondition
> - Expected postcondition
> - Error condition







## Correctness arguments

> Proof sketches of why we believe the solution satisfies the problem statement.
Possibly giving inductive invariants that can be used to prove the specifications
of the problem statement 

# References

> links to other specifications/ADRs this document refers to


[block]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockcha
in.md
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
