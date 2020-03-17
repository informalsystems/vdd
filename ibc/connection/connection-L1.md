# IBC Connection


## 1. Problem Statement / Outside view

****************D E S C R I P T I O N*************************************

This part of the specification should present the problem at the very
high level (if possible using sequential specification). We can think
about this part of the specification as an operational level of the
problem where basic mechanisms are explained, but without
going into details about the particular system model in which problem
can be solved and without entering protocol design details.

This part of the specification serves as a gentle introduction to the
problem. It is useful to both protocol designers and engineers as a high
level problem description.

**************************************************************************

IBC connection is a protocol that ensures establishment of connection
between two modules. At the end of the connection handshake at both
modules there is common view on the corresponding data structures.

One of the party (called sometimes source or module B) is triggering
connection establishment and the other module follows.

Properties that should hold:

- data structures have consistent view at the end of the handshake
- protocol eventually terminates


#### Expected Expertise

- **lead:** distributed algorithm designer: Adi
- **input/feedback:** distributed systems engineers

#### Artifacts

  - High level English specification of the problem:
  ibc-rs/docs/spec/connection/L1_2.md

#### Verification, Validation, or Proof Obligation

- N.A.

### 2. Protocol Specification / Protocol view

****************D E S C R I P T I O N*************************************

This part of the specification should present concrete system model
in which problem is considered, and an algorithm that solves the problem
in the given model. It contains two parts: 1) system model specification
and 2) algorithm (protocol) specification.

**************************************************************************

#### 2.1 System Model Specification

****************D E S C R I P T I O N*************************************

System model section should therefore contains the assumptions we made
regarding the following aspects:

- definitions of processes (process represents unit of execution) involved.
A process can be a node in a distributed system or a thread (routine)
in case of concurrent applications. Furthermore, we need to specify what
kind of process faults we consider: no faults, crash-stop faults,
crash-recovery faults, arbitrary (Byzantine) faults, etc. In addition to
kind of faults we need also to specify a maximum bound (number) of faulty
processes we assume: less than majority, less than one third, all
connected processes could be faulty, etc.
- definitions of communications between processes: message passing,
shared memory, remote procedure calls, channels (blocking,
unblocking, bounded, etc), etc.
- synchrony assumptions on the process and network speed: synchronous
(there is a known upper bound on the process/network speed),
asynchronous (no assumption is made on upper bound on the process/network
speed), partially synchronous (system in between synchronous and
asynchronous, i.e., system is eventually synchronous, or transitions between
periods of asynchrony and synchrony).
- safety and liveness properties of the problem in the given model
(these are the properties that an algorithm must fulfil to be able to
solve the problem in the given system model).

****************S Y S T E M  M O D E L*************************************

- Processes: We assume processes running on different (distributed)
hosts. A process is normally called module in this contect. A module is
a deterministic state machine that change its state only be
executing transactions. In a typical setup, a host on which module is
running is part of the replicated state machine (blockchain). Host can
also be a single node.

Processes can be faulty: we assume that a process can behave completely
arbitrary (Byzantine faults). There is no upper bound on the number
of faulty processes.

- Communications: processes communicate by exchanging messages. A module
A is sending a message m to a module B by writing it to a local storage
(data store), i.e., it is not directly sending it over wire to the module
B. We then assume that there is an external process (relayer) that is
reading messages that are sent by A and deliver it to the corresponding
module. We assume that message m sent by a correct module A to a correct
module B will be eventually delivered to the module B. Sending a message
is not a blocking call, i.e., module continues execution after sending
a message. We assume that every module has a unique identifier,
and that every message contains cryptographic signature that cannot be
forged, i.e., if a module B receives message m from the module A, then
m was sent by the module A before.

- Synchrony and ordering assumptions: we assume an asynchronous communication:
if a correct module A send a message m to a correct module B, the
message m is eventually received by the module B. We don't assume that
messages will be delivered in the order of sending, i.e., network can
reorder messages.

****************P R O P E R T I E S***************************************

Safety 1: Once connection is established between correct processes,
the corresponding data structures contains the same data (identifiers,
commitment prefixes, initial consensus state).

Safety 2: Faulty module cannot prevent correct modules from establishing
connection.

Safety 3: Faulty module cannot execute man in the middle attack.

Liveness 1: If a correct process A triggers connection establishment to
a correct process B, then the connection is eventually established.


##### Expected Expertise

  - **lead:** distributed algorithm designer: Adi
  - **input/feedback:** distributed systems engineers,
  verification engineers

##### Artifacts

  - High-level English specification covering aspects mentioned above
  (processes, communication channels, synchrony assumptions, safety and
  liveness properties): ibc-rs/docs/spec/connection/L1_2.md

##### Verification, Validation, or Proof Obligation

  - Informal check that the problem refinement in the given model
  corresponds to the top level problem definition.

#### 2.2 Algorithm (Protocol) Specification

****************D E S C R I P T I O N*************************************

This part presents an algorithm (protocol) that solves the problem
in the system model specified in the section 2.1. Algorithm specification
at this level should be at the higher level of abstractions
compared to real implementation, and should be seen mainly as part of
the protocol design phase. High level algorithm specification should cover:

- messages exchanged (minimal set of messages needed to express
algorithm core algorithm logic)
- core data structures every process maintains (minimal data structures
needed to express core algorithm logic).
- state machine(s) that defines algorithm transitions and
- protocol invariants.

****************P R O T O C O L*******************************************

We should consider a single instance of the Connection Handshake protocol,
where a correct module A triggers establishment of connection with a
module B. Therefore, we assume that nodes manages only a single instance
of core data structures.

Messages:

- ConnInit
- ConnOpenTry
- ConnOpenAck
- ConnOpenConfirm

Data structures:

- ConnectionEnd -- state, connectionId, clientId, counterParty, versions
- CounterParty -- connectionId, clientId, commitmentPrefix

We probably don't need this at the high level:

- LightClient -- consensusState
- ConsensusState -- height, rootHash

We assume also the following helper functions (provided by environment):

- getCurrentHeight -- returns current height (at the local blockchain)
- getClientConsensusState(height) -- returns consensus state for the given
height of the counter party module


- verifyConnectionState -- verifies that the message about counter party
connection end is valid. Verification is done by relying on light client
verification mechanism.
- verifyClientConsensusState -- verifies that the message about counter
party perception of local consensus state is valid.

The protocol will be expressed as two party distributed protocol. Processes
are denoted with A and B (maybe we should use Alice and Bob as in crypto
protocols). Protocol starts when a process A receives ConnInit message.
Protocol proceeds by processes exchanging messages (ConnOpenTry, ConnOpenAck
and ConnOpenConfirm) and updating local ConnectionEnd. Every message
conveys information about local process state and its correctness is
verified at the counter party using helper functions. Note that
the high level protocol ignores details about underlying light client
updates; at this level we assume that it is provided by the execution
environment through the helper functions. Furthermore, the relayer
functionality is abstracted away at this level through the properties
of the communication layer.

Sketch of one message handler:

State of every process

connectionEnd -- initially NilConnection

function connOpenInit(
  identifier: Identifier,
  clientIdentifier: Identifier,
  counterParty: CounterParty) {
    if connectionEnd != NilConnection then error

    connectionEnd.connectionIdentifier = identifier
    connectionEnd.clientIdentifier = clientIdentifier
    connectionEnd.state = INIT
    connectionEnd.counterParty = counterParty

    consensusStateB = getConsensusState(connectionEnd.clientIdentifier)

    send ConnOpenTry(
        connectionEnd, getProof(connectionEnd),
        consensusStateB, getProof(consensusStateB))
}

function connOpenTry(
        connectionEnd_A,
        proofOfConnectionEnd,
        consensusStateB,
        proofOfConsensusStateB) {

  if connectionEnd != NilConnection then error

  verifyConnectionState(proofOfConnectionEnd, connectionEnd_A)
  verifyClientConsensusState(proofOfConsensusStateB, consensusStateB)

  connectionEnd = createConnectionEnd(connectionEnd_A)
  connectionEnd.state = TRYOPEN

  consensusStateA = getConsensusState(connectionEnd.clientIdentifier)

  send ConnOpenAck(
        connectionEnd, getProof(connectionEnd),
        consensusStateA, getProof(consensusStateA))

}


##### Expected Expertise

  - **lead:** Adi, Zarko
  - **input/feedback:** verification engineers and distributed systems
  engineers

##### Artifacts

  - English description of the protocol: ibc-rs/docs/spec/connection/L1_2.md
  - TLA+ specification of the protocol: ibc-rs/docs/spec/connection/L2.tla
  - TLA+ specification of properties (expressed in English in the section
  2.1) as invariants and temporal properties: ibc-rs/docs/spec/connection/L2.tla

##### Verification, Validation, or Proof Obligation

  - Check that the protocol satisfies the invariants and temporal logic
  properties


### 3. Single Node View

****************D E S C R I P T I O N*************************************

At this level we think about the specification of the code. Even in
distributed systems, code inherently runs on one computer, so we take the
single node perspective here while the rest of the
system is modelled as the environment.

****************L O W  L E V E L  S P E C*********************************

Connection Handshake protocol is implemented in practice by implementing
multiple nodes (roles):

- IBC connection module
- Relayer
- IBC light client module

IBC connection module implements core connection handshake protocol
from the endpoint view point. It is a deterministic state machine
that receives input events, update state and writes it to the data store.

Relayer is a process (could contain multiple threads of execution)
that ensures the following:

- secure communication between modules
- supports light client mechanisms (light client updates).

As relayer is also a distributed protocol, we will treat it in this
context as a black box that provides communication guarantees and
the relayer should be treated the same way (complete VDD spec) in a
separate document.

Light client is a process that ensures based on a provides trusted state,
how to update trusted state without participating in blockchain consensus
protocol. It is a distributed protocol on its own, and we will consider
it as part of this protocol as an oracle that provides some services.


#### Expected Expertise

- **lead: Ilina
- **input/feedback:** distributed protocol designer, verification engineer

#### Artifacts

- Code of the IBC connection module (currently Go implementation)
- TLA+ specification of the IBC connection module (multi connection support,
data store, light client management, error handling, etc):
ibc-rs/docs/spec/connection/L3.tla
- TLA+ specification of properties introduced at the
implementation level (for example concurrency related) expressed as
invariants and temporal properties:
ibc-rs/docs/spec/connection/L3.tla

#### Verification, Validation, or Proof Obligation

- Check that the specification satisfies the invariants and temporal logic
  properties
- TLA+ reductions / refinement mappings with respect to the high level
specification
- Check that code passes tests that corresponds to (automatically) generated
abstract execution scenarios.
