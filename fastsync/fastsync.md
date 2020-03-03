

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
node was disconnected.

## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view. 

A full node has as input a prefix of the current blockchain until
height *h*, and a list of full nodes called *peers* that it knows of.
The full node reads blocks of the Tendermint blockchain, until it has
read the most recent block and then terminates.


## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

*Fastsync* gets as input a block of height *h*, and produces as
outpout a list of blocks.

#### **[FS-Seq-Live]**: 
*Fastsync* eventually terminates.
 
#### **[FS-Seq-Term]**:
When *Fastsync* terminates, it outputs a list of all blocks from
height *h* to the current height of the blockchain [**[TMBC-SEQ]**][TMBC-SEQ-link].

*Remark:* this will require timing assumptions on the rate at which a
block is added to the blockchain, and message and computation delays
involved in Fastsync.

#### **[FS-Seq-Inv]**:
*Fastsync* never stores a header which is not in the blockchain.



# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects 

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly 

> should have clear formalization in temporal logic.

Peers can be faulty, and we do not make any assumption about number or
ratio of correct/faulty nodes.

#### **[FS-A-Comm]**:
Communication between Fastsync and a correct full node is reliable and bounded in time.

#### **[LFS-A-LCC]**:
The node executing Fastsync is following the protocol (it is correct).

TODO: message passing vs. rpc


## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

They exchange the following messages:

- StatusRequest
- StatusReply
- BlockRequest
- BlockReply


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

### Outline

> Describe solution (in English), decomposition into functions, where communication to other components happens.

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
