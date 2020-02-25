*** This is the beginning of an unfinished draft. Don't continue reading! ***

# Tendermint Full Node

> Rough outline of what the component is doing and why. 2-3 paragraphs 

# Part I - Outside view

A full node provides an API to query specific information from the blockchain.

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc. 

> should give the reader the understanding in what environment this component
will be used. 

TODO: copy remotely exposed functions here, and the corresponding invariants,
e.g., SignedHeaders returned by commit

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

### Inter Process Communication



We assume that the Tendermint Full Node exposes the following functions over Tendermint RPC:

```go
func Commit(height int64) (SignedHeader, error)
```
- Implementation remark
   - RPC to full node _n_
- Expected precodnition
  - header of `height` exists on blockchain
- Expected postcondition
  - if _n_ is correct: Returns the signed header of height `height`
  from the blockchain if communication is timely (no timeout)
  - if _n_ is faulty: Returns a signed header with arbitrary content
- Error condition
   * if _n_ is correct: precondition violated or timeout
   * if _n_ is faulty: arbitrary error

----


 ```go    
func Validators(height int64) (ValidatorSet, error)
```
- Implementation remark
   - RPC to full node _n_
- Expected precodnition
  - header of `height` exists on blockchain
- Expected postcondition
  - if _n_ is correct: Returns the validator set of height `height`
  from the blockchain if communication is timely (no timeout)
  - if _n_ is faulty: Returns arbitrary validator set
- Error condition
  - if _n_ is correct: precondition violated or timeout 
  - if _n_ is faulty: arbitrary error

----

#### **[FN-LuckyCase]**:
The primary is correct and no timeout occurs on `Commit` and `Validators`.


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
