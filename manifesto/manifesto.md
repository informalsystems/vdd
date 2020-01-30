# Verification-Driven Development: An Informal Manifesto

**Abstract.** Software bugs can be increasingly costly, e.g., blockchain
technology. At the same time the software is quite complex. For instance,
all-in-all the Tendermint protocols are written in x lines of Golang code.

Computer-aided verification is impractical, either because completely automated
verification methods (such as model checking) hit theoretical limits and
undecidability results, or because methods based on interactive theorem provers
require a huge amount of manual work and understanding of the protocols. Part of
the problem is that verification is often regarded as a task that happens "after
the fact", that is, when the software is written.

We are developing an approach where verification and software
design and development go hand-in-hand. Our goal is to improve software
engineering practices by relying on formal methods. Our approach focuses on
fault-tolerant distributed systems.


## Introduction

- because fault-tolerance, we have to deal with time and faults


## Development Work Flow

In this section we discuss the steps we take starting with rather high-level informal requirements to formal requirements (in temporal logic), a protocol description, etc. to arrive at an efficient implementation of a distributed protocol.

This process is a collaboration of researchers, verification engineers, and distributed systems and software engineers. However, different steps require different expertise in the lead of this step. We make these requirements explicit.

The whole process will be thoroughly documented by artifacts like English specifications, TLA+ specifications, and code artifacts including tests.

### 1. Problem Statement / Outside view:

<< this part gives a very high level view

Abstract sequential specification. << needs more discussion >>

how this works together with other specs

#### Points addressed

  - The problem that we would like to solve, the involved parties
  - probably in a sequential view:
    e.g.,

        - the blockchain is a growing list,
        - a lite client is just a node which gets an integer h as input.
        - The goal is to return the data that is stored at height h of the block chain

#### Expected Expertise

- **lead:** distributed algorithm designer
- **input/feedback:**

  << this might be super abstract, and only make sense in combination with the Protocol in the beginning. >>

#### artifacts

  - Part I of high-level English spec.
  - TLA+ Spec of temporal properties (in case we want to do TLA+ reductions)

#### Verification, Validation, or Proof Obligation

- perhaps composition with other specifications



### 2. Protocol Specification / Protocol view:

As we focus on fault-tolerant distributed systems, we have to specify protocols that run on unreliable/adversarial
computers and networks, that refine the problem statement from above.

#### Points addressed

  - environment << perhaps put this in a new point 2 >>
      - the blockchain is implemented by a set of validators and other full nodes
      - validators may be Byzantine
      - message passing
      - asynchrony, time...

  - first level of solution

  - What does it mean for the distributed system to solve the abstract problem statement? refinement of problem statement: e.g.;

        - in case of at most 1/3 faulty validators safety property A and liveness B
        - in case of between 1/3 and 2/3 then safety property A' and liveness B'
        - if the liteclient is connected to a correct full node and communication is
           timely, then C and D

#### Expected Expertise

  - **lead:** distributed algorithm designer
  - **input/feedback:** distributed systems engineers

#### Artifacts

  - Part II of high-level English spec.
  - TLA+ Spec of the protocol
  - TLA+ json
  - TLA+ Spec of the temporal properties (the problem solved)
  - refined Problem Statement (adapt Section 1)

#### Verification, Validation, or Proof Obligation

- Simulation relation / refinement. How we map events/executions in a faulty and distributed environment to the ideal specification from Point 1.
- check refinement for small systems in Apalache or TLC?
- check that the protocol satisfies the refined temporal logic specifications
- TLA+ reductions / refinement mappings


### 3. API Specification / Single Node View

At this level we think about the specification of the code.
Code inherently runs on one computer, so we take the
single node perspective here
and interactions with the rest of the systems is modeled as environment.

#### Points addressed

From a single point perspective, we capture

 - API (signatures)
 - Mapping from TLA+ states and transitions (protocol specification Point 2)
   to the API (atomic propositions, input/output events).
   Mapping the functions of the API to in-out events in TLA+ spec of protocol: TLA+ json to API json transformation
 - Data structures
 - Error Handling

all this must be linked to the TLA+ protocol specs from above

 - Tests that abstract the environment

#### Expected Expertise
- **lead:** software engineer, adversarial engineer
- **input/feedback:**

#### Artifacts

- English Implementation Spec << do we need that as a "text document"? wouldn't a code artifact be more suitable?>>

- ADR?
- API
- Rust traits
- API json
- test driver: produces tests over Rust traits from API json
- hand-written tests


#### Verification, Validation, or Proof Obligation

- generate tests with Apalache and TLC
- check consistency of API << todo: think about suitable tool >>

### 4. Prototype Implementation / Code View

#### Points addressed

In this step we implement the first functional version of the software

#### Expected Expertise
- **lead:** software engineer
- **input/feedback:**

#### artifacts

- Software


#### Verification, Validation, or Proof Obligation

- Running the software against the Tests

- Understand performance bottlenecks


### 5. Implementation

For performance reasons we typically need to decompose the program logic into concurrent tasks. We have
to ensure that by doing so, we maintain invariants  and temporal logic specifications in general.



#### Points addressed

- concurrency architecture of the implementation.
- invariants and temporal formulas that express the correctness of the concurrency architecture (how it refines the simpler implementation from Point 4)
- need to check whether Point 2 or Point 3 (Protocol Specification or Implementation Specification) needs to be reconsidered, e.g., if the concurrency requires more involved protocols within the single node perspective.


#### Expected Expertise
- **lead:** software engineer
- **input/feedback:**

#### artifacts

- code
- possibly new tests focussing on concurrency aspects

#### Verification, Validation, or Proof Obligation

- Running the software against the Tests
