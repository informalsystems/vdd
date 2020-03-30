
# Fastsync

<!--
> Rough outline of what the component is doing and why. 2-3 paragraphs 
---->

# Part I - Outside view

## Context of this document

<!--
> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc. 
---->
<!--
> should give the reader the understanding in what environment this component
will be used. 
---->
Fastsync is a protocol that is used by a full node to catch-up to the
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

<!--
> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view. 
---->
A full node has as input a block of the blockchain at height *h* and
the corresponding application state (or the prefix of the current
blockchain until height *h*). It has access to a set *peerIDs* of full
nodes called *peers* that it knows of.  The full node uses the peers
to read blocks of the Tendermint blockchain (in a safe way, that is,
it checks the soundness conditions), until it has read the most recent
block and then terminates.


## Sequential Problem statement

<!--
> should be English and precise. will be accompanied with a TLA spec.
---->
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
[**[TMBC-SEQ]**][TMBC-SEQ-link].




#### **[FS-Seq-Inv]**:
Upon termination, the application state is the one that corresponds to
the blockchain at height *terminationHeight*.




# Part II - Protocol view

## Environment/Assumptions/Incentives

<!--
> Introduce distributed aspects 
---->
<!--
> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly 
---->
<!--
> should have clear formalization in temporal logic.
---->


#### **[FS-A-NODE]**:
We consider a node *FS* that performs *Fastsync*.

#### **[FS-A-PEER-IDS]**:
*FS* has access to a set *peerIDs* of IDs (public keys) of peers (full
     nodes). During execution of *Fastsync* another protocol (outside
     of this specification) may add new IDs to *peerIDs*.



#### **[FS-A-PEER]**:
Peers can be faulty, and we do not make any assumption about number or
ratio of correct/faulty nodes. Faulty processes may be Byzantine
according to [**[TMBC-Auth-Byz]**][TMBC-Auth-Byz-link].

#### **[FS-A-VAL]**:
The system satisfies [**[TMBC-Auth-Byz]**][TMBC-Auth-Byz-link] and [**[TMBC-FM-2THIRDS]**][TMBC-FM-2THIRDS-link]. Thus, there is a
blockchain that satisfies the soundness requirements [**[TMBC-SOUND-?]**][blockchain].

#### **[FS-A-COMM]**:
Communication between *FS* and all correct peers is reliable and
bounded in time: there is a message end-to-end delay *Delta* such that
if a message is sent at time *t* by a correct process to a correct
process, then it will be received and processed by time *t +
Delta*. This implies that we need a timeout of at least *2 Delta* for
remote procedure calls to ensure that the response of a correct peer
arrives before the timeout expires.

<!--
#### **[FS-A-LCC]**:
The node *FS* executing Fastsync is following the protocol (it is correct).
---->

## Distributed Problem Statement

### Design choices

<!--
> input/output variables used to define the temporal properties. Most likely they come from an ADR
---->

We do not put assumptions on the existence of a correct full node
in *peerIDs*. Under this assumption we cannot guarantee the properties
described in the sequential specification above. Thus, in the (unreliable)
distributed setting, we consider two kinds of termination (normal and
abort) and we will specify below under what (favorable) conditions we
can ensure to terminate normally:

#### **[FS-DISTR-TERM]**:
*Fastsync* may *terminate normally* or it  *aborts*.


#### Remote Functions

The Tendermint Full Node exposes the following functions over
Tendermint RPC. The "Expected precondition" are only expected for
correct peers (as no assumption is made on internals of faulty
processes [FS-A-PEER]).


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
  - if *addr* is faulty: Returns an arbitrary height. [**[TMBC-Auth-Byz]**][TMBC-Auth-Byz-link]
- Error condition
   * if *addr* is correct: none. By [FS-A-COMM] we assume communication is reliable and timely.
   * if *addr* is faulty: arbitrary error (including timeout). [**[TMBC-Auth-Byz]**][TMBC-Auth-Byz-link]
----


 ```go
func Block(addr Address, height int64) (Block, error)
```
- Implementation remark
   - RPC to full node *addr*
- Expected precondition
  - header of `height` is less than or equal to height of the peer
- Expected postcondition
  - if *addr* is correct: Returns the block of height `height`
  from the blockchain. [FS-A-COMM]
  - if *addr* is faulty: Returns arbitrary block [**[TMBC-Auth-Byz]**][TMBC-Auth-Byz-link]
- Error condition
  - if *addr* is correct: precondition violated. [FS-A-COMM]
  - if *addr* is faulty: arbitrary error (including timeout). [**[TMBC-Auth-Byz]**][TMBC-Auth-Byz-link]
----

### Temporal Properties


#### Fairness

We sometimes consider the following (fairness) constraint in the
safety and liveness properties below:


#### **[FS-CORR-PEER]**:
At all times, the set *peerID* contains at least one correct full node.

#### **[FS-ALL-CORR-PEER]**:
At all times, the set *peerID* contains only correct full nodes.


#### Safety


<!--
> safety specifications / invariants in English 
---->


#### **[FS-VC-ALL-CORR-NONABORT]**:
Under [FS-ALL-CORR-PEER], *Fastsync* never aborts.


#### **[FS-VC-INV]**:
If *FastSync* terminates normally at height *terminationHeight*, then the
application state is the one that corresponds to the blockchain at
height *terminationHeight*.


#### **[FS-VC-CORR-INV]**:
Under [FS-CORR-PEER], let *t* be the maximum 
height of a correct peer [**[TMBC-CorrFull]**][TMBC-CorrFull-link]
in *peerIDs* at the time *Fastsync* starts. If *FastSync* terminates
normally, it is at some height *terminationHeight >= t*.


#### Liveness

<!--
> liveness specifications in English. Possibly with timing/fairness requirements:
e.g., if the component is connected to a correct full node and communication is
reliable and timely, then something good happens eventually. 
---->

#### **[FS-VC-ALL-CORR-TERM]**:
Under [FS-ALL-CORR-PEER], *Fastsync* eventually terminates normally.


<!--
> How is the problem statement linked to the "Sequential Problem statement". 
Simulation, implementation, etc. relations 
---->


## Definitions

<!--
> In this section we become more concrete, with basic (abstracted) data types 
---->
<!--
> some math that allows to write specifications and pseudo code solution below.
Some variables, etc. 
---->



#### Inputs
- *startBlock*: the block Fastsync starts from
- *startState*: application state corresponding to *startBlock.Height*

#### **[FS-A-V2-INIT]**:
- *startBlock* is from the blockchain
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

#### Auxiliary Functions

#### **[FS-FUNC-TARGET]**:
- *TargetHeight = max {peerHeigts(addr): addr in peerIDs} union {startBlock.height}*

#### **[FS-FUNC-MATCH]**:

**TODO:** what is the name of the Golang function that implements
that?

```go
func CommitMatchesBlock(a Block, b Block) Boolean
```
- Implementation remark
    - implements the check from
     [**[TMBC-SOUND-DISTR-PossCommit]**][TMBC-SOUND-DISTR-PossCommit--link],
     that is, that  *b.Commit* is a valid commit for block *a*
- Expected precondition
    - *b.Commit* is a valid commit for block *a*
- Expected postcondition
    - *true* if precondition holds
	- *false* if precondition is violated
- Error condition
    - none
----

#### **[FS-VAR-STATE-INV]**:
It is always the case that *state* corresponds to the application state of the
blockchain of that height, that is, *state = chain[height].AppState*
[**[TMBC-SEQ]**][TMBC-SEQ-link].

#### **[FS-VAR-PEER-INV]**:
It is always the case that the set *peerIDs* only contains nodes that
have not yet misbehaved (by sending wrong data or timing out).





## Solution

<!--
> Basic data structures. Simplified, so that we can focus on the distributed
algorithm here. If existing: link to Tendermint data structures, and mentioned
if details were omitted. 
---->

### Outline

<!--
> Describe solution (in English), decomposition into functions, where communication to other components happens.
---->

The protocol is described in terms of functions that are triggered by
(external) events. The implementation typically uses a scheduler and a
de-multiplexer to deal with communicating with peers and to
trigger the execution of these functions:

- `QueryStatus()`: regularly (currently every 10sec; necessarily
  interval greater than *2 Delta*) queries all peers from *peerIDs*
  for their current height [TMBC-LOCAL-CHAIN]. It does so
  by calling `Status(n)` remotely on all peers *n*.
  
- `CreateRequest`: regularly checks whether certain blocks have no
  open request. If a block does not have an open request, it requests
  one from a peer. It does so by calling `Block(n,h)` remotely on one
  peer *n* for a missing height *h*.
  

The functions `Status` and `Block` are called by asynchronous
RPC. When they return, the following functions are called:

- `OnStatusResponse(addr Address, height int64)`: The full node with
  address *addr* returns its current height. The function updates the height
  information about *addr*, and may also increase *TargetHeight*.
  
- `OnBlockResponse(addr Address, b Block)`. The full node with
  address *addr* returns a block. It is added to *blockstore*. Then
  the auxiliary function `Execute` is called.

- `Execute()`: Iterates over the *blockstore*.  Checks soundness of
  the blocks, and
  executes the transactions of a sound block and updates *state*. If
  the longest prefix reaches *TargetHeight* it terminates *Fastsync*.
  
During execution *peerIDs* may become empty. In this case *Fastsync*
aborts.

**TODO:** is the above termination condition OK?

### Details

<!--
> Function signatures followed by pseudocode (optional) and a list of features (required):
> - Implementation remarks (optional)
>   - e.g. (local/remote) function called in the body of this function
> - Expected precondition
> - Expected postcondition
> - Error condition
---->

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
func OnStatusResponse(addr Address, ht int64)
```
- Comment
    - *ht* is a height
- Expected precondition
    - *peerHeights(addr) <= ht*
- Expected postcondition
    - *peerHeights(addr) = ht*
	- *TargetHeight* is updated
- Error condition
    - if precondition is violated: *addr* not in *peerIDs* (that is,
      *addr* is removed from *peerIDs*)
- Timeout condition
    - if `OnStatusResponse(addr, ht)` was not invoked within *2 Delta* after
	`Status(addr)` was called:  *addr* not in *peerIDs*
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
    - if function `Execute` has been executed without error
        - *receivedBlocks(b.Height) = addr*
	    - *blockstore(b.Height) = b*
- Error condition
    - if precondition is violated: *addr* not in *peerIDs*; reset
	*pendingblocks(b.Height)* to nil;
- Timeout condition
    - if `OnBlockResponse(addr, b)` was not invoked within *2 Delta* after
	`Block(addr,h)` was called for *b.Height = h*: *addr* not in *peerIDs*
----



```go
func Execute()
```
- Comments
    - none
- Expected precondition
	- application state is the one of the blockchain at height *height*
- Expected postcondition
    - **[FS-V2-Verif]** for any two blocks *a* and *b* from *receivedBlocks*, with
	  *a.Height + 1 = b.Height*: if *CommitMatchesBlock (a,b) = false*, then
	  *a* and *b* not in *blockstore*; nodes with Address 
	  receivedBlocks(a.Height) and receivedBlocks(b.Height) not in peerIDs
	- height is updated height of complete prefix that matches the blockchain
	- state is the one of the blockchain at height *height*
	- **[FS-V2-TERM]** if height = TargetHeight: **terminate normally**
- Error condition
    - none
----

## Analysis

####  **[FS-ISSUE-KILL]**:
If two blocks are not matching, the rule [FS-V2-Verif] dismisses both
blocks and removes the peers that provided these blocks from
*peerIDs*. If block *a* was correct and provided by a correct peer *p*,
and block b was faulty and provided by a faulty peer, the protocol
- removes the correct peer *p*, although it might be useful to
  download blocks from it in the future
- removes *a*, so that a fresh copy of *a* needs to be downloaded
  again from another peer
  
By [FS-A-PEER] we do not put a restriction on the number
  of faulty peers, so that faulty peers can make *FS* to remove all
  correct peers from *peerIDs*


####  **[FS-ISSUE-NON-TERM]**:

By [FS-A-PEER-IDS], peers may be added to *peerIDs*. By [FS-A-PEER],
we do not put a restriction on the number of faulty peers, so that it
is possible that faulty peers are added to *peerIDs* at arbitrary
rate.  Every time `QueryStatus()` is called, *FS* invokes `Status(n)`
for a faulty node *n*. This node may return an arbitrary height, in
particular one that is greater than the current height of the
blockchain. As a result, it is possible that permanently
*TargetHeight* is greater than the height of the blockchain, so that
the termination condition in [FS-V2-TERM] is never reached.


### Consequence

Due to [FS-ISSUE-KILL] and [FS-ISSUE-NON-TERM], the temporal
properties that are relevant for termination, namely, 
[FS-VC-ALL-CORR-TERM] and [FS-VC-ALL-CORR-NONABORT], need to be
restricted to the case where all peers are correct
[FS-ALL-CORR-PEER]. In a fault tolerance context this is problematic,
as it means that faulty peers can prevent *FastSync* from termination.

## Possible Fix

### Temporal Properties

Instead of the limited termination properties [FS-VC-ALL-CORR-TERM]
and [FS-VC-ALL-CORR-NONABORT], a fault-tolerant solution shall satisfy
the following two properties:

#### **[FS-VC-NONABORT]**:
If there is one correct process in *peerIDs* [FS-CORR-PEER],
*Fastsync* never aborts. (Together with [FS-VC-TERM] below that means
it will terminate normally.)

#### **[FS-VC-TERM]**:
*Fastsync* eventually terminates normally or it eventually aborts.


### Solution for [FS-ISSUE-KILL]

To avoid [FS-ISSUE-KILL], we observe that
[**[TMBC-FM-2THIRDS]**][TMBC-FM-2THIRDS-link] ensures that from the
point a block was created, we assume that more than two thirds of the
validator nodes are correct until the *trustingPeriod* expires.  Under
this assumption, assume the trusting period of *startBlock* is not
expired by the time *FastSync* checks a block *b* with height
*startBlock.Height + 1*.  Even if *CommitMatchesBlock (startBlock,b) =
false*, it is clear that *startBlock* is OK, and the peer *p* that
provided block *b* is faulty. Only peer *b* should be removed from
*peerIDs*. That is, if we sequentially verify blocks starting with
*startBlock*, we will never remove a correct peer from *peerIDs* and
we will be able to ensure the following invariant:


#### **[FS-VAR-PEER-INV]**:
If a peer never misbehaves, it is never removed from *peerIDs*. It
follows that under [FS-CORR-PEER], *peerIDs* is always non-empty.
To perform these checks, we suggest to change the protocol as follows


#### Fastsync has the following configuration parameters:
- *trustingPeriod*: a time duration
  [**[TMBC-TIME_PARAMS]**](TMBC-TIME_PARAMS-link).

[FS-A-INIT] is the suggested replacement of [FS-A-V2-INIT]. This will
allow us to use the established trust to understand precisely which
peer reported an invalid block in order to ensure the
invariant [FS-VAR-TRUST-INV] below:

#### **[FS-A-INIT]**:
- *startBlock* is from the blockchain, and within *trustingPeriod*
(possible with some extra margin to ensure termination before
*trustingPeriod* expired)
- *startState* is the application state of the blockchain at Height
  *startBlock.Height*.
- *startHeight = startBlock.Height*

#### Additional Variables
- *trustedBlockstore*: stores for each height greater than
    *startBlock.Height*, the block of that height. Initially it
    contains only *startBlock*


#### **[FS-VAR-TRUST-INV]**:
Let *b(i)* be the block in *trustedBlockstore*
with b(i).Height = i. It holds that
for *startHeight <= i < height*, 
*CommitMatchesBlock (b(i),b(i+1)) = true*.



Then we propose to update the function `Execute`. To do so, we first
define the following helper function:


```go
func SequentialVerify {
	for i, bnew range receivedBlocks {
	    // We assume receivedBlocks is iterates in order of block Heights
		if bnew.Height == height + 1 {
		    if CommitMatchesBlock (trustedBlockstore[height], bnew) == true {
			    trustedBlockstore.Add(bnew);
			    height = bnew.Height;
			}
			else {
			    blockstore.RemoveFromPeer(receivedBlocks(bnew.Height));
				// we remove all blocks received from the faulty peer
			    peerIDs.Remove(receivedBlocks(bnew.Height));
				exit;
			}
		}
	}
}
```
- Comments
    - none
- Expected precondition
	- [FS-VAR-TRUST-INV]
- Expected postcondition
    - [FS-VAR-TRUST-INV]
	- there is no block *bnew* with *bnew.Height = height + 1* in
      *blockstore*
- Error condition
    - none 
----


Then `Execute` just consists in calling `SequentialVerify` and then
updating the application state to the (new) height.

```go
func Execute()
```
- Comments
    - first `SequentialVerify` is executed
- Expected precondition
	- application state is the one of the blockchain at height *height*
- Expected postcondition
    - there is no block *bnew* with *bnew.Height = height + 1* in
      *blockstore*
	- state is the one of the blockchain at height *height*
	- if height = TargetHeight: **terminate normally**
- Error condition
    - none 
----


### Solution for [FS-ISSUE-NON-TERM]

As discussed above, the advantageous termination requirement is the
combination of [FS-VC-NONABORT] and [FS-VC-TERM], that is, *Fastsync*
should terminate normally in case there is at least one correct
peer in *peerIDs*.

#### **[FS-SOLUTION-TERM-BOUND]**:

A simple way to achieve this is to assume that there is an a priori
fixed upper bound *t* on the number of faulty peers. Then, we could
change the definition of *TargetHeight*: instead of picking the
maximum received height [FS-FUNC-TARGET], *TargetHeight* could be the
"fault-tolerant maximum", that is the t+1 largest height reported by
peers. This ensures that the *TargetHeight* is less than or equal to a
height reported by a correct peer and the scenario in
[FS-ISSUE-NON-TERM] can be avoided.

The downside of this is that *t* is "hard-coded" into the solution,
and if the assumption on *t* is violated liveness also is violated.


#### **[FS-SOLUTION-TERM-GOOD]**:

Here we assume that during a "long enough" finite good period no new
faulty peers are added to *peerIDs*. Below we will sketch how "long
enough" can be estimated based on the timing assumption in this
specification. 

#### **[FS-A-GOOD-PERIOD]**:

A time interval *[begin,end]* is *good period* if:

- *fmax* is the number of faulty peers in *peerIDs* at time *begin*
- *end >= 2 Delta (fmax + 3)*
- no faulty peer is added before time *end*


#### **[FS-A-GOOD-PERIOD-FASTSYNC]**:

Arguments: 

1. If a faulty peer *p* reports a faulty block, `SequentialVerify` will
  eventually remove *p* from *peerIDs*
  
2. By `SequentialVerify`, if a faulty peer *p* reports multiple faulty
  blocks, *p* will be removed upon trying to check the block with the
  smallest height received from *p*.

3. Assume whenever a block does not have an open request, `CreateRequest` is
   called immediately, which calls `Block(n)` on a peer. Say this
   happens at time *t*. There are two cases: 
  
   - by t + 2 Delta a block is added to *blockStore*
   - at t + 2 Delta `Block(n)` timed out and *n* is removed from
       peer.
	   
4. Let *f(t)* be the number of faulty peers in *peerIDs* at time *t*;  
   *f(begin) = fmax*.
	   
5. Let t_i be the sequence of times `OnBlockResponse(addr,b)` is
   invoked or times out with *b.Height = height + 1*.
   
6. By 3., 
   - (a). *t_1 <= begin + 2 Delta* 
   - (b). *t_{i+1} <= t_i + 2 Delta* 

7. By an inductive argument we prove for *i > 0* that

   - (a). *height(t_{i+1}) > height(t_i)*, or
   - (b). *f(t_{i+1}) < f(t_i))* and *height(t_{i+1}) = height(t_i)*  

   Argument: if the peer is faulty and does not return a block, the
   peer is removed, if it is faulty and returns a faulty block
   `SequentialVerify` removes the peer (b). If the returned block is OK,
   height is increased (a).
	 
8. By 2. and 7., faulty peers can delay incrementing the height at
   most *fmax* times, where each time "costs" *2 Delta* seconds. We
   have additional *2 Delta* initial offset (3a) plus *2 Delta* to get
   all missing blocks after the last fault showed itself. (This
   assumes that an arbitrary number of blocks can be obtained and
   checked within one round-trip 2 Delta; which either needs
   conservative estimation of Delta, or a more refined analysis). Thus
   we reach the *targetHeight* and terminate by time *end*.


# References

<!--
> links to other specifications/ADRs this document refers to
---->

[block]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md

[blockchain]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md

[TMBC-HEADER-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-header

[TMBC-SEQ-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-seq

[TMBC-CorrFull-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-corrfull

[TMBC-Sign-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-sign

[TMBC-FaultyFull-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-faultyfull

[TMBC-TIME_PARAMS-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-time_params

[TMBC-FM-2THIRDS-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-fm-2thirds

[TMBC-Auth-Byz-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-auth-byz

[TMBC-INV-SIGN-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-inv-sign

[TMBC-SOUND-DISTR-PossCommit--link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-sound-distr-posscommit

[TMBC-INV-VALID-link]: https://github.com/informalsystems/VDD/tree/master/blockchain/blockchain.md#tmbc-inv-valid

[LCV-VC-LIVE-link]: https://github.com/informalsystems/VDD/tree/master/lightclient/verification.md#lcv-vc-live

[lightclient]: https://github.com/interchainio/tendermint-rs/blob/e2cb9aca0b95430fca2eac154edddc9588038982/docs/architecture/adr-002-lite-client.md

[failuredetector]: https://github.com/informalsystems/VDD/blob/master/liteclient/failuredetector.md

[fullnode]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md

[FN-LuckyCase-link]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md#fn-luckycase

[blockchain-validator-set]: https://github.com/tendermint/spec/blob/master/spec/blockchain/blockchain.md#data-structures

[fullnode-data-structures]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md#data-structures

[FN-ManifestFaulty-link]: https://github.com/tendermint/spec/blob/master/spec/blockchain/fullnode.md#fn-manifestfaulty
