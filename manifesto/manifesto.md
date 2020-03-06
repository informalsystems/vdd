# Verification-Driven Development: An Informal Manifesto

**Abstract.** Software bugs can be increasingly costly, e.g., blockchain
technology. At the same time, modern software stacks are quite complex.
For instance, Byzantine fault tolerant blockchain systems(
for example Tendermint) are based on complex fault-tolerant distributed
protocols, implemented as highly concurrent systems, and overall contains
thousands lines of code. Therefore, there are lot of space for potential
bugs: protocol bugs, concurrency bugs, implementation bugs, security bugs, etc.

As bugs in (some) modern distributed systems has potentially high cost,
people has started considering computer-aided verification tools and
methodologies as a way to increase correctness and verifiability of
those critical systems. However current approaches turns out to be
impractical (to large scale), either because of tools and methodology
theoretical limits (completely automated verification methods such as
model checking hit theoretical limits and undecidability results), or
because methods (approaches based on interactive theorem provers) require
a huge amount of manual work.

Part of the problem is the fact that the verification is often regarded
as "after the fact" task, that is, used after the software is written.
In this document we present an approach where verification, software
design and development go hand-in-hand. Our goal is to improve software
engineering practices (and ultimately the end software product)
by relying on formal methods. Our approach primarily focuses on
distributed and concurrent systems, although it might be of interest also
outside those domains.

## Development Work Flow

In this section we discuss the steps, deliverables and responsibilities
in the process of designing and developing distributed/concurrent system.
This process is a collaboration of researchers (protocol and/or verification
experts) and software engineers. Different steps require different expertise
in the lead of that step. We make these requirements explicit.

### 1. Problem Statement / Outside view:

This part of the specification should present the problem at the very
high level (if possible using sequential specification). We can think
about this part of the specification as an operational level of the
problem where basic mechanisms are explained, but without
going into details about the particular system model in which problem
can be solved and without entering protocol design details.

This part of the specification serves as a gentle introduction to the
problem. It is useful to both protocol designers and engineers as a high
level problem description.

#### Expected Expertise

- **lead:** distributed algorithm designer
- **input/feedback:** distributed systems engineers

#### Artifacts

  - High level English specification of the problem.
  - [Optional] In some cases it might be useful to write TLA+
  specification of temporal properties (in case we want to do TLA+
  reductions), but this is by default not required

#### Verification, Validation, or Proof Obligation

- We don't require any proofs or formal validation to be done at this
level. In case temporal properties are expressed in TLA+, it can be
perhaps used in composition with other specifications.

### 2. Protocol Specification / Protocol view:

This part of the specification should present concrete system model
in which problem is considered, and an algorithm that solves the problem
in the given model. It contains two parts: 1) system model specification
and 2) algorithm (protocol) specification.

#### 2.1 System Model Specification

As we focus on (fault-tolerant) distributed and concurrent systems, we
have to specify protocols that run on unreliable/adversarial
computers and networks, that refine the problem statement from above.

System model section should therefore contains the assumption we made
about following aspects:

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
speed, partially synchronous (system in between synchronous and
asynchronous, i.e., system is eventually synchronous, or transitions between
periods of asynchrony and synchrony).
- safety and liveness properties of the problem in the given model
(these are the properties that an algorithm must fulfil to be able to
solve the problem in the given system model).

##### Expected Expertise

  - **lead:** distributed algorithm designer
  - **input/feedback:** distributed systems engineers,
  verification engineers

##### Artifacts

  - High-level English specification covering aspects mentioned above (
  processes, communication channels, synchrony assumptions, safety and
  liveness properties.

##### Verification, Validation, or Proof Obligation

  - Informal check that the problem refinement in the given model
  corresponds to the top level problem definition.

#### 2.2 Algorithm (Protocol) Specification

This part presents an algorithm (protocol) that solves the problem
in the system model specified in the section 2.1. Algorithm specification
at this level should be at the higher level of abstractions
compared to real implementation, and should be seen mainly as part of
protocol design phase. High level algorithm specification should cover:

- messages exchanged (minimal set of messages needed to express
algorithm core algorithm logic)
- core data structures every process maintains (minimal data structures
needed to express core algorithm logic)
- state machine(s) that defines algorithm transitions and
- protocol invariants.

High level protocol specifications should be at the higher level of abstractions
compared to real implementation, and it should not introduce concepts that could differ
between implementations. For example, high level protocol specifications
should (if possible) avoid dealing explicitly with timeouts or
efficiency concerns (batching of messages, flow control logic,
DDoS protection mechanisms, etc), concurrency aspects, detailed error
handling, etc. Some of those concerns might be addressed at the lower
specification levels, and some would just appear at the implementation level.

Depending on the protocol type, we might need to model complete
distributed system with a set of processes and communication channels
between them (for example consensus) as safety and liveness properties
are global (it is about all correct processes). In other cases, where
protocol is more single node oriented (for example fast sync, state sync,
light client, etc), it might be sufficient modelling a single node
(that is service consumer) and all other processes as a single environment.
This should simplify modelling and make model checking more efficient.
It also makes it simpler to consider multiple environments to be able to
model weaker or stronger adversaries in a more modular way.


##### Expected Expertise

  - **lead:** distributed algorithm designer and/or verification engineer
  - **input/feedback:** verification engineers and distributed systems
  engineers

##### Artifacts

  - TLA+ specification of the protocol
  - TLA+ specification of properties (expressed in English in the section
  2.1) as invariants and temporal properties
  - Abstract test scenarios: generation of (relevant) abstract
  execution scenarios (using model checkers) that can be used to
  drive unit and integration tests.
  - [Optional] In addition to properties that corresponds to protocol
  safety and liveness properties, good practice is also writing a set
  of simpler failing properties that are useful for having more
  certainty in the correctness of the specification (that concepts are
  correctly encoded) and also as a mean of generating interesting
  test cases (by the model checker). These properties should be prefixed
  with Failing, for example ```FailingPeerSetIsNeverEmpty```.

##### Verification, Validation, or Proof Obligation

  - Check that the protocol satisfies the invariants and temporal logic
  properties
  - [Optional] TLA+ reductions / refinement mappings in case higher level
  specifications are expressed in TLA+.


### 3. Single Node View

At this level we think about the specification of the code. Even in
distributed systems, code inherently runs on one computer, so we take the
single node perspective here and interactions with the rest of the
system is modeled as environment.

The goal of specification at this level (we call it also low level
specification) is to be as close as possible to the implementation.
Being close to the implementation allows discovery of software
architecture problems and potentially implementation issues.
Depending on the implementation complexity the specification at this
level should contain:

- state machine of the single process that maps high level protocol
transitions to the node API (input/output events). This state machine
should normally contain more complex data structures (compared to
the one from the section 2.2), new concepts and mechanisms
(implementation specific) that could be either efficiency motivated or
consequence of programming language environment, and detailed error
handling logic.
- multiple state machines in case a node implementation actually compose
of multiple concurrent processes (tasks). In this case in addition to
the elements already mentioned (the single task case), we also need to
model concurrency architecture of the solution, defining additional
invariants and temporal properties for the concurrency architecture
(for example that the solution does not have deadlocks). Note that in
this case, we might need to write multiple TLA+ specifications
that will be analysed and model checked in isolation, together with the
complete solution that would correspond to the overall node implementation.


#### Expected Expertise

- **lead:** distributed systems engineer
- **input/feedback:** distributed protocol designer, verification engineer

Note that at this stage of development we expect engineers to have a lead
by proposing implementation informed by the high level specification
(the section 2) and formal specification and verification should be seen
as a supportive tools to discover implementation level design issues,
discover bugs, generate interesting abstract test scenarios; in general
to increase confidence in the code correctness.

Although in general, engineers are not constrained in the way system
will be implemented, we expect verification engineers to inform
some implementation decisions from the verifiability perspective.
Some known good engineering practices that align well with formal tools
are:

- modular design (reduce complexity by splitting implementation into
smaller modules with clear and simple responsibilities that can be
designed, implemented, tested and verified in isolation)
- functional programming (core business logic should be expressed as a
function (for example state machine) which based on input state and events
generate new state and output events, i.e., avoid side effects like
state mutation)
- simpler concurrency architectures (concurrent tasks are owners of
particular concepts and communication between tasks happen explicitly
by exchanging events, i.e., avoid communication between different tasks
over shared data and locks)

#### Artifacts

- Code
- TLA+ specification of the state machines that correspond to code artefact
- TLA+ specification of properties introduced at the
implementation level (for example concurrency related) expressed as
invariants and temporal properties
- Abstract test scenarios: generation of (relevant) abstract
execution scenarios (using model checkers) that can be used to
drive unit and integration tests.
- [Optional] Simpler failing properties that are useful for having more
certainty in the correctness of the specification (that concepts are
correctly encoded) and also as a mean of generating interesting
test cases (by the model checker). These properties should be prefixed
with Failing.

#### Verification, Validation, or Proof Obligation

- Check that the specification satisfies the invariants and temporal logic
  properties
- TLA+ reductions / refinement mappings with respect to the high level
specification
- Check that code passes tests that corresponds to (automatically) generated
abstract execution scenarios.

