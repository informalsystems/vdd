*** This is the beginning of an unfinished draft. Don't continue reading! ***

# Tendermint Blockchain

> Rough outline of what the component is doing and why. 2-3 paragraphs

A blockchain is a growing sequence of transactions called log. If several processes in a distributed system have access to the blockchain, (1) they can  provide transactions as input and (2) they can use the log to execute these transactions in order. This then ensures the state machine replication.

The log itself should be implemented in a reliable way, which introduces the need for fault-tolerance and distribution.
Tendermint implements state machine replication over an unreliable (Byzantine) distributed system. More precisely, Tendermint provides a replicated log where each log entry contains several transactions.

In this specification, we are only considered with the logs. They are maintained at so-called full nodes, and are the result of iterated execution of the Tendermint consensus protocol. Each iteration, upon deciding on the transactions to be put into the log, creates a new log entry. The iterations are numbered and are called heights.


# Part I - Outside view

## Context of this document

> mention other components and or specifications that are relevant for this
spec. Possible interactions, possible use cases, etc.

> should give the reader the understanding in what environment this component
will be used.

This specification is in the core of the set of Tendermint protocol. The behavior of protocols like fastsync (**link**), or the lite client (**link**) will be defined with respect to this specification. E.g., the lite client implements a read operation of the log entry of height _h_. It is thus crucial to understand what data is stored in this log entry, and what it means that a read is successful in a faulty environment.


## Informal Problem statement

> for the general audience, that is, engineers who want to get an overview over what the component is doing
from a bird's eye view.


## Sequential Problem statement

> should be English and precise. will be accompanied with a TLA spec.

**[TMBC-SEQ]** The Tendermint blockchain is a list of headers with heights:

 -  Heights are strictly increasing.

 - If the blockchain contains a header of height *h*, then for all _h'<h_
it contains a header of height _h'_.

 - During operation, new headers may one-by-one be appended to the list.

# Part II - Protocol view

## Environment/Assumptions/Incentives

> Introduce distributed aspects

> Timing and correctness assumptions. Possibly with justification that the
assumptions make sense, e.g., it is in the interest of a full node to behave
correctly

> should have clear formalization in temporal logic.

There are bad people in the world... proof of stake ...

validators, full nodes,

**[TMBC-FM-2THIRDS]** If a block _h_ is generated at time _bfttime_ (and this time is stored in the block), then a set of validators that hold more than 2/3 of the voting power in h.Header.NextV is correct until time h.Header.bfttime + tp.

Formally,
\[
\sum*{(v,p) \in h.Header.NextV \wedge correct(v,h.Header.bfttime + tp)} p >
2/3 \sum*{(v,p) \in h.Header.NextV} p
\]

_Assumption_: "correct" is defined w.r.t. realtime (some Newtonian global notion of time, i.e., wall time), while _bfttime_ corresponds to the reading of the local clock of a validator (how this time is computed may change when the Tendermint consensus is modified). In this note, we assume that all clocks are synchronized to realtime. We can make this more precise eventually (incorporating clock drift, accuracy, precision, etc.). Right now, we consider this assumption sufficient, as clock synchronization (under NTP) is in the order of milliseconds and _tp_ is in the order of weeks.





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
