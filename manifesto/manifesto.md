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

### 1. Problem Statement:


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

  - Section 1 of high-level English spec.
  - TLA+ Spec (in case we want to do TLA+ reductions)

#### Verification, Validation, or Proof Obligation

- perhaps composition with other specifications


### 2. Protocol Specification

As we focus on fault-tolerant distributed systems, we have to specify protocols that run on unreliable/adversarial
computers and networks, that refine the problem statement from above.

#### Points addressed

  - first level of solution
  - introduce the full-blown distribution: e.g.;

        - the blockchain is implemented by a set of validators and other full nodes
        - validators may be Byzantine
        - there is a node running the lite client
        - the lite client and the full nodes communicate by message passing
        - asynchrony, time...

  - What does it mean for the distributed system to solve the abstract problem statement? refinement of problem statement: e.g.;

        - in case of at most 1/3 faulty validators safety property A and liveness B
        - in case of between 1/3 and 2/3 then safety property A' and liveness B'
        - if the liteclient is connected to a correct full node and communication is
           timely, then C and D

#### Expected Expertise

  - **lead:** distributed algorithm designer
  - **input/feedback:** distributed systems engineers

#### artifacts

  - Section 2 of high-level English spec.
  - TLA+ Spec
  - refined Problem Statement (adapt Section 1)

#### Verification, Validation, or Proof Obligation

- Simulation relation / reduction. How we map events/executions in a faulty and distributed environment to the ideal specification from Point 1.
- check refinement for small systems in Apalache or TLC?
- check that the protocol satisfies the refined temporal logic specifications
- TLA+ reductions


### 3. Implementation Specification

At this level we think about the specification of the code. Code inherently runs on one computer, so we take the
single node perspective here
and interactions with the rest of the systems is modelled as environment.

#### Points addressed

From a single point perspective, we capture

 - API
 - Data structures
 - Error Handling
 - Tests that abstract the environment

#### Expected Expertise
- **lead:** software engineer, adversarial engineer
- **input/feedback:**

#### Artifacts

- English Implementation Spec
- ADR?
- API
- Rust traits
- tests
- automatically generated counterexamples



#### Verification, Validation, or Proof Obligation



### 4. Prototype Implementation

#### Points addressed

In this step we implement the first functional version of the software

#### Expected Expertise
- **lead:** software engineer
- **input/feedback:**

#### artifacts

- Software


#### Verification, Validation, or Proof Obligation

- Running the software against the Tests

- Understand performance bottle necks


### 5. Implementation

For performance reasons we typically need to decompose the program logic into concurrent tasks. We have
to ensure that by doing so, we maintain invariants  and temporal logic specifications in general.

We might want to look at the perspective
of each concurrent task and to model its behavior and see if invariants hold. Furthermore,
at the level of each task there might be additional invariants we want to add so
corresponding test scenarios will be executed.


#### Points addressed

- concurrency architecture of the implementation.
- need to check whether Point 2 or Point 3 (Protocol Specification or Implementation Specification) needs to be reconsidered, e.g., if the concurrency requires more involved protocols within the single node perspective.

#### Expected Expertise
- **lead:** software engineer
- **input/feedback:**

#### artifacts



#### Verification, Validation, or Proof Obligation

- Running the software against the Tests




## Lessons learned
