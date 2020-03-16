
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
recovering full node then queries its peers for the blocks that were
decided on by the Tendermint blockchain during the period the full
node was disconnected. It then executes the transactions in the block
in order to catch-up to the current height of the blockchain and the
corresponding application state.

## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view. 

A full node has as input a block of the blockchain at height *h* and
the corresponding application state (or the prefix of the current
blockchain until height *h*). It has access to a set of full nodes
called *peers* that it knows of.  The full node uses the peers to read
blocks of the Tendermint blockchain (in a safe way, that is, checking
the soundness conditions), until it has read the most recent block and
then terminates.


## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

*Fastsync* gets as input a block of height *h* and the corresponding
application state *s*, and produces as
output a list *l* of blocks and the application 
state when applying the transaction
of the list *l* to *s*.

#### **[FS-Seq-Live]**: 
*Fastsync* eventually terminates.

*Remark:* this will require timing assumptions on the rate at which a
block is added to the blockchain, and message and computation delays
involved in Fastsync. That is, the time it takes fastsync to append a
block to the list, and do the verification and the computation of the
application state should be less than *ETIME* from [TMBC-SEQ-APPEND-E].
 
#### **[FS-Seq-Term]**:
When *Fastsync* terminates, it outputs a list of all blocks from
height *h* to the current height *h'* of the blockchain,
[**[TMBC-SEQ]**][TMBC-SEQ-link].  
Upon termination the application state is the one that corresponds to
*h'*.



#### **[FS-Seq-Inv]**:
*Fastsync* never stores a header/block which is not in the blockchain.



# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects 

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly 

> should have clear formalization in temporal logic.

**TODO:** peer exchange maintains a list of peers (full nodes)

Peers can be faulty, and we do not make any assumption about number or
ratio of correct/faulty nodes.

Full nodes satisfy [TMBC-Auth-Byz] [TMBC-FM-2THIRDS]

#### **[FS-A-Comm]**:
Communication between Fastsync and a correct peer is reliable and
bounded in time.

#### **[FS-A-LCC]**:
The node executing Fastsync is following the protocol (it is correct).

TODO: message passing vs. rpc


## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR


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

> safety specifications / invariants in English 

> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually. 

should have clear formalization in temporal logic.

> How is the problem statement linked to the "Sequential Problem statement". 
Simulation, implementation, etc. relations 

## Definitions

> In this section we become more concrete, with basic (abstracted) data types 

> some math that allows to write specifications and pseudo code solution below.
Some variables, etc. 

## Solution

> Basic data structures. Simplified, so that we can focus on the distributed
algorithm here. If existing: link to Tendermint data structures, and mentioned
if details were omitted. 

#### Fastsync has the following configuration parameters:
- *trustingPeriod*: a time duration [**[TMBC-TIME_PARAMS]**](TMBC-TIME_PARAMS-link).
- *clockDrift*: a time duration. Correction parameter dealing with
  only approximately synchronized clocks.
  

#### Interface to peer exchange
- *peerIDs*: peer addresses provided by peer exchange

**Question1:** if a peer is removed does peer exchange figure that our?

#### Inputs
- *startBlock*: block of height Fastsync starts from
- *startState*: state corresponding to startBlock.Height

#### Variables
- *height*: initially *startBlock.Height*
- *state*: initially *startState*
- *peerHeigts*: stores for each peer the height it reported. initially 0
- *pendingBlocks*: stores for each height which peer was
  queried. initially nil
- *receivedBlocks*: stores for each height which peer returned it
- *blockstore*: stores for each height greater than
    *startBlock.Height*, the block of that height

#### Definitions
- *TargetHeight = max {peerHeigts(addr): addr in peerIDs}*  
  *Remark:* it is only computed over peers that are not considered
  faulty, yet

### Outline

> Describe solution (in English), decomposition into functions, where communication to other components happens.

Initially, all 

```go
func QueryStatus() {
  for i, s range peerIDs {
    sh := Status(s)
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
    - *peerHeights(addr) <= height* ?
- Expected postcondition
    - *peerHeights(addr) = height*
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
    - Block is called at a peer in peerIDs for a missing height 
	- if height = TargetHeight: stop everything
- Error condition
    - if *peerIDs* is empty: no correct peers left; panic
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
    - if precondition is violated: remove *addr* from *peerIDs*
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
	- state is the one of the blockchain at height *heigt*
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
