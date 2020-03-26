I think that I have some idea how we can capture fast-sync safety property in a model with adding peers and continuos status requests. The idea is to say that in case correct process terminates at some time t, then he downloaded/executed at least all blocks that are advertised(received status messages) by correct processes until time t-T1, where T1 is terminationTimeout (plus maybe some additional Delta). So we essentially say that you can’t terminate if you haven’t fetched all correct processes advertised. This is main safety property and termination property will need to be specified by constraining addPeer messages. I think  that protocol also needs to be organised more synchronously as a sequence of rounds, where a round is basically what we currently have in TLA+ spec.

*** This is the beginning of an unfinished draft. Don't continue reading! ***

# << Name of Component >>

> Rough outline of what the component is doing and why. 2-3 paragraphs 

# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc. 

> should give the reader the understanding in what environment this component
will be used. 

## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view. 


## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects 

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly 

> should have clear formalization in temporal logic.

## Distributed Problem Statement

### Design choices

> input/output variables used to define the temporal properties. Most likely they come from an ADR

### Temporal Properties



#### **[FS-VC-NONABORT]**:
Under [FS-CORR-PEER], *Fastsync* never aborts. (Together with
[FS-VC-TERM] below that means it will terminate normally.)

#### **[FS-VC-TERM]**:
*Fastsync* eventually terminates normally or it eventually aborts.


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

#### Fastsync has the following configuration parameters:
- *trustingPeriod*: a time duration
  [**[TMBC-TIME_PARAMS]**](TMBC-TIME_PARAMS-link).



#### **[FS-A-INIT]**:
- *startBlock* is from the blockchain, and within *trustingPeriod*
(possible with some extra margin to ensure termination before
*trustingPeriod* expired)
- *startState* is the application state of the blockchain at Height
  *startBlock.Height*.  

[FS-A-INIT] is the suggested replacement of [FS-A-V2-INIT]. This will
allow us to use the established trust to understand precisely which
peer reported an invalid block in order to ensure the following
invariant:
  
  
#### **[FS-VAR-PEER-INV]**:
If a peer never misbehaves, it is never removed from *peerIDs*. It
follows that under [FS-CORR-PEER], *peerIDs* is always non-empty.


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


**TODO:** explain sequential checks

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
	not in *blockstore*; node with Address
	receivedBlocks(b.Height) not in peerIDs
----



## Correctness arguments

> Proof sketches of why we believe the solution satisfies the problem statement.
Possibly giving inductive invariants that can be used to prove the specifications
of the problem statement 

# References

> links to other specifications/ADRs this document refers to
